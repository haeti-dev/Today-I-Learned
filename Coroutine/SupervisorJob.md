## Job vs SupervisorJob

- `Job`의 기본 동작은 “one-for-all, all-for-on” 정책을 적용한다.
    - 스코프 내 자식 코루틴이 예외로 인해 실패하면 부모 코루틴을 즉시 취소하고, 부모 코루틴은 다른 모든 형제 코루틴을 취소한다.
    - 엄격한 구조화된 동시성을 적용한다. 실패를 위쪽으로 전파된다.
- `SupervisorJob`은 이러한 경직된 장애 전파를 깨기 위해 설계된 특수한 유형의 Job이다.
    - 하위 Job들이 Supervisor Job 자체나 **다른 하위 Job들에 영향을 미치지 않고 독립적으로 장애를 일으킬 수 있는 범위를 생성**하는 것이다.
    - 이를 통해 장애 격리가 가능해지며, 이는 하나의 장애 작업으로 전체 시스템이 중단되지 않는 앱을 구축하는데 중요한 패턴이다.
    - 실패는 위쪽으로 전파되지 않는다. 부모 SupervisorJob이나 다른 자식에는 영향을 미치지 않고, 실패는 해당 자식으로만 국한된다.

## SupervisorJob의 내부 메커니즘

```kotlin
// In a regular JobSupport instance
public open fun childCancelled(cause: Throwable): Boolean {
    // A CancellationException is considered normal, don't cancel the parent.
    if (cause is CancellationException) return true

    // For any other exception, cancel the parent itself.
    return cancelImpl(cause)
}
```

- JobSupport에서 자식 코루틴이 실패하면 결국 부모의 ChildHandle에 childCancelled(cause)를 호출한다. 표준 `Job`에서 이 메서드의 기본 구현은 실패를 위쪽으로 전파하도록 설계되었다.
- 이것이 실패 전략이다. 자식 프로세스에서 취소 불가 예외가 발생하면 부모 프로세스가 스스로 취소된다.

```kotlin
@Suppress("FunctionName")
public fun SupervisorJob(parent: Job? = null) : CompletableJob = SupervisorJobImpl(parent)

private class SupervisorJobImpl(parent: Job?) : JobImpl(parent) {
    override fun childCancelled(cause: Throwable): Boolean = false
}
```

- SupervisorJobImpl을 인스턴스화하는 간단한 팩토리가 있다.
- 여기서는 이 `childCancelled`를 항상 false로 오버라이드하고 있다. 이로써 “자식이 실패를 알려주었지만, 아직 예외를 처리하지 않았고, 이 때문에 스스로 취소하지 않을 것이다” 라고 이 시스템에 전달할 수 있다.
- 이렇게 하면 오류의 상향 전파가 효과적으로 차단된다. `SupervisorJob`은 활성 상태를 유지하고 다른 자식 프로세스들은 영향을 받지 않고 계속 실행된다.
- 실패한 자식 프로세스의 예외 처리 책임은, 자식 프로세스 컨텍스트 내의 `CoroutineExceptionHandler` 또는 `async`, `Deferred` 결과의 소비자에게 위임된다.

### supervisorScope builder

```kotlin
public suspend fun <R> supervisorScope(block: suspend CoroutineScope.() -> R): R {
    // ...
    val coroutine = SupervisorCoroutine(uCont.context, uCont)
    // ...
}

private class SupervisorCoroutine<in T>(...) : ScopeCoroutine<T>(...) {
    override fun childCancelled(cause: Throwable): Boolean = false
}
```

- `SupervisorJobImpl` 과 마찬가지로 `SupervisorCoroutine` `childCancelled` 재정의하여 `false` 반환하는 특수 코루틴이다.
- `supervisorScope` 블록 내에서 새 코루틴을 실행하면 부모 `Job` 이 `SupervisorCoroutine` 이 된다. 따라서 해당 자식 코루틴에서 발생하는 모든 실패는 `SupervisorCoroutine` 에서 중단되어 실패가 외부로 유출되는 것을 방지하고 `supervisorScope` 호출한 외부 범위를 취소한다.

### 사용 사례

- viewModelScope가 대표적이다. ViewModel에서 진행되는 작업들은 대게 서로 독립적이며 서로를 중단시키면 안되는 경우가 많다.

```kotlin
public val ViewModel.viewModelScope: CoroutineScope
    get() =
        synchronized(VIEW_MODEL_SCOPE_LOCK) {
            getCloseable(VIEW_MODEL_SCOPE_KEY)
                ?: createViewModelScope().also { scope ->
                    addCloseable(VIEW_MODEL_SCOPE_KEY, scope)
                }
        }
        
internal fun createViewModelScope(): CloseableCoroutineScope {
    val dispatcher =
        try {
            Dispatchers.Main.immediate
        } catch (_: NotImplementedError) {
            EmptyCoroutineContext
        } catch (_: IllegalStateException) {
            EmptyCoroutineContext
        }
    return CloseableCoroutineScope(coroutineContext = dispatcher + SupervisorJob()) // here!
}
```

- 내부 코드를 보면 SupervisorJob을 사용하고 있다.
