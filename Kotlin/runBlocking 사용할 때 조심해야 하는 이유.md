## **Android에서 runBlocking 사용할 때 조심해야 하는 이유**

### **Kotlin의 비동기 프로그래밍과 runBlocking**

코루틴을 실행하는 여러 가지 방법이 있지만, **runBlocking**은 실행하는 동안 현재 스레드를 멈춰버린다.

공식 문서에서도 자주 등장하는데, **Android에서는 신중하게 써야 한다.**


<br>

**왜 runBlocking이 Android에서 문제일까?**

<br>


Android의 **메인 스레드**는 UI를 그리는 중요한 역할을 한다. runBlocking은 현재 스레드가 끝날 때까지 기다리게 때문에, runBlocking을 메인 스레드에서 실행하면 **앱이 멈추거나 ANR(Application Not Responding) 오류**가 발생할 수 있다.

<br>


### runBlocking 내부 살펴보기

```kotlin
public actual fun <T> runBlocking(
    context: CoroutineContext, 
    block: suspend CoroutineScope.() -> T
): T {
    val currentThread = Thread.currentThread()
    val contextInterceptor = context[ContinuationInterceptor]
    val eventLoop: EventLoop?
    val newContext: CoroutineContext

    if (contextInterceptor == null) {
        eventLoop = ThreadLocalEventLoop.eventLoop
        newContext = GlobalScope.newCoroutineContext(context + eventLoop)
    } else {
        eventLoop = (contextInterceptor as? EventLoop)
            ?.takeIf { it.shouldBeProcessedFromContext() }
            ?: ThreadLocalEventLoop.currentOrNull()
        newContext = GlobalScope.newCoroutineContext(context)
    }

    val coroutine = BlockingCoroutine<T>(newContext, currentThread, eventLoop)
    coroutine.start(CoroutineStart.DEFAULT, coroutine, block)
    return coroutine.joinBlocking()
}
```

- runBlocking은 현재 스레드에서 새로운 코루틴을 실행한다.
- **GlobalScope를 활용해 코루틴 컨텍스트를 설정**한다.
- **코루틴이 끝날 때까지 현재 스레드를 계속 차단한다.**

<br>


### **BlockingCoroutine 내부 동작**

특히, runBlocking이 코루틴을 실행할 때 내부적으로 사용하는 BlockingCoroutine의 joinBlocking 메서드를 보면, 이 동작이 확실해진다.

<br>


```kotlin
private class BlockingCoroutine<T>(
    parentContext: CoroutineContext,
    private val blockedThread: Thread,
    private val eventLoop: EventLoop?
) : AbstractCoroutine<T>(parentContext, true, true) {

    fun joinBlocking(): T {
        registerTimeLoopThread()
        try {
            eventLoop?.incrementUseCount()
            try {
                while (true) {
                    if (Thread.interrupted()) throw InterruptedException().also { cancelCoroutine(it) }
                    val parkNanos = eventLoop?.processNextEvent() ?: Long.MAX_VALUE
                    if (isCompleted) break
                    parkNanos(this, parkNanos)
                }
            } finally {
                eventLoop?.decrementUseCount()
            }
        } finally {
            unregisterTimeLoopThread()
        }

        val state = this.state.unboxState()
        (state as? CompletedExceptionally)?.let { throw it.cause }
        return state as T
    }
}
```

- joinBlocking() 내부를 보면 **while (true) 무한 루프**가 돌고 있다.
- **현재 스레드를 차단한 채 계속 이벤트를 처리**하고 있다.
- **코루틴이 끝날 때까지 무한 루프를 유지**하며, 그동안 **스레드는 아무 작업도 못 한다.**

<br>


결국 **이게 메인 스레드에서 실행되면 UI가 멈춰버리는 이유다.**

## 예제를 통해 문제상황 살펴보기

### **📌 예제 1: Dispatchers.IO에서 runBlocking 실행**

```kotlin
fun sample1() = runBlocking(Dispatchers.IO) {
    val currentThread = Thread.currentThread()
    Log.d("Main", "Current Thread: $currentThread")
    delay(3000) // 3초 동안 대기
    Log.d("Main", "Job Completed")
}
```

✅ Dispatchers.IO 덕분에 백그라운드 스레드에서 실행된다.

❌ 하지만 **메인 스레드는 여전히 차단된다.** 결국 **UI가 멈추는 문제**가 발생할 수 있다.

<br>


### **📌 예제 2: Dispatchers.Main에서 runBlocking 실행**

```kotlin
fun sample2() = runBlocking(Dispatchers.Main) {
    val currentThread = Thread.currentThread()
    Log.d("Main", "Current Thread: $currentThread")
    delay(3000) // 3초 대기
    Log.d("Main", "Job Completed")
}
```

🚨 **이 코드는 실행되지 않는다.**

- runBlocking(Dispatchers.Main)이 실행되면 **메인 스레드가 차단**된다.
- 근데, Dispatchers.Main은 **메인 스레드에서 실행되려면 스레드가 비어 있어야 하는데, 이미 차단된 상태**라서 데드락(Deadlock)이 발생한다.
- 결과적으로 **앱이 영원히 멈춰버린다.**

<br>


### **📌 예제 3: runBlocking을 백그라운드 스레드에서 실행**

```kotlin
fun sample3() = CoroutineScope(Dispatchers.IO).launch {
    val result = runBlocking {
        delay(3000)
        "Completed"
    }
    Log.d("Main", "Result: $result")
}
```

✅ 이번엔 백그라운드 스레드에서 실행했기 때문에 **메인 스레드는 영향을 받지 않는다.**

⚠️ 하지만 **백그라운드 스레드를 차단하는 것도 좋은 방법은 아니다.**

⚠️ 보통 이런 작업은 **비동기 방식으로 처리하는 게 더 낫다.**

<br>


## **그럼 runBlocking은 언제 써도 괜찮을까?**

메인 스레드에서 runBlocking을 쓰면 위험하지만, 특정한 경우엔 써도 괜찮다.

<br>


### **1. 단위 테스트(Unit Test)**

```kotlin
@Test
fun testFunction() = runBlocking {
    val result = fetchData() // suspend 함수
    assertEquals(expectedValue, result)
}
```

- 테스트 환경에서는 메인 스레드를 차단해도 문제없다.
- 근데 `runTest`를 쓰는 게 더 좋은 방법이다.

<br>


### **2. IO 스레드에서 동기 작업(Synchronization)**

```kotlin
fun syncTask() = runBlocking(Dispatchers.IO) {
    val result = fetchData()
    Log.d("Main", "Result: $result")
}
```

- IO 작업을 할 때 runBlocking을 쓰면 결과를 기다릴 수 있다.
- 하지만 가능하면 **job.join()을 쓰는 게 더 안전하다.**

<br>


## **🚀 runBlocking 대신 쓸 수 있는 더 좋은 방법**

**✅ 1. launch + join 사용**

```kotlin
fun alternative() = CoroutineScope(Dispatchers.IO).launch {
    val job = launch {
        delay(3000)
        Log.d("Main", "Task Completed")
    }
    job.join() // 해당 작업이 끝날 때까지 대기
    Log.d("Main", "Execution Resumed")
}
```

✅ runBlocking 없이도 **코루틴이 끝날 때까지 기다릴 수 있다.**

✅ **메인 스레드를 차단하지 않는다.**

<br>


### **결론**

1. runBlocking을 **메인 스레드에서 실행하면 앱이 멈출 수 있다.**
2. Dispatchers.IO에서 실행한다고 해도 **스레드를 차단하는 동작은 여전히 존재한다.**
3. **단위 테스트나 백그라운드 작업에서만 조심해서 써야 한다.**
4. **가능하면 job.join() 같은 비동기 처리 방법을 쓰는 게 더 낫다.**

### 레퍼런스
https://getstream.io/blog/caution-runblocking-android/
