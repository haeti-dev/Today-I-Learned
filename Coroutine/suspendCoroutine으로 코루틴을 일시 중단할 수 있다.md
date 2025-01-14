## suspendCoroutine이란?

`suspendCoroutine`은 코루틴을 일시 중단하고, 결과가 준비되면 재개할 수 있게 해주는 함수이다. 주로 콜백 기반 API를 코루틴 스타일로 변환할 때 사용된다.

## 실제 사용 예시

다음은 네트워크 요청을 수행하는 가상의 콜백 기반 API를 코루틴으로 변환하는 예시이다.

```kotlin
// 기존의 콜백 기반 API
class NetworkClient {
    fun fetchData(onSuccess: (String) -> Unit, onError: (Exception) -> Unit) {
        // 비동기로 데이터를 가져오는 작업
        Thread {
            Thread.sleep(1000) // 네트워크 지연 시뮬레이션
            onSuccess("데이터 전송 완료")
        }.start()
    }
}

// suspendCoroutine을 사용한 코루틴 스타일 변환
class NetworkRepository(private val client: NetworkClient) {
    suspend fun fetchData(): String = suspendCoroutine { continuation ->
        client.fetchData(
            onSuccess = { data ->
                continuation.resume(data)
            },
            onError = { exception ->
                continuation.resumeWithException(exception)
            }
        )
    }
}

```

## 핵심 개념 정리

### suspendCoroutine

- 코루틴을 일시 중단하고 결과를 기다릴 수 있게 해주는 함수
- 콜백 기반 API를 코루틴 스타일로 변환할 때 사용
- 람다 내부에서 `continuation` 객체를 제공받음

### Continuation

- 코루틴의 상태를 나타내는 인터페이스
- 코루틴이 중단된 지점의 정보를 가지고 있음
- `resume()` 또는 `resumeWithException()`을 통해 코루틴을 재개할 수 있음

## 배운 점

1. `suspendCoroutine`을 사용하면 콜백 기반 코드를 더 읽기 쉬운 코루틴 스타일로 변환할 수 있다.
2. `Continuation` 객체는 코루틴의 상태를 관리하고 제어하는 핵심 요소이다.
3. `resume()`을 통해 비동기 작업의 결과를 코루틴에 전달할 수 있다.
