![image](https://github.com/user-attachments/assets/31ce016d-feca-4dbd-8d33-cf85f2ddd10e)

<br/>

`channelFlow`는 “동시에(concurrently) 데이터가 생산될 수 있는 흐름”을 만들 때 사용한다. 보통 `callbackFlow`와 비슷한 용도로 쓰이지만, 중요한 차이점 중 하나는 **여러 코루틴 컨텍스트**에서 메시지를 안전하게 전송할 수 있고, 코루틴 간 병렬 실행을 가능하게 한다는 점이다.

- **Cold Flow**: channelFlow로 만든 Flow는 *cold* 속성을 가진다. 즉, 최종 오퍼레이터(예: collect)가 호출될 때마다 내부의 block이 새로 실행된다.
- **Thread-safety & Context preservation**: channelFlow는 스레드 세이프하며, 제공된 ProducerScope를 여러 코루틴이 동시에 사용해도 문제가 없도록 설계되었다.
- **버퍼(Buffer)**: 내부적으로 Channel.BUFFERED 버퍼를 사용한다. 필요 시 buffer(size) 오퍼레이터를 통해 버퍼 사이즈나 백프레셔(back-pressure) 전략을 조정할 수 있다.
- awaitClose **활용**: 콜드 플로우이기 때문에, 비동기적인 콜백을 계속 받고 싶다면 awaitClose { ... }를 이용해 블록이 살아있도록 해야 한다.

<br/>


### 예시

```kotlin
fun <T> Flow<T>.merge(other: Flow<T>): Flow<T> = channelFlow {
    // 하나의 코루틴에서 first Flow 수집
    launch {
        collect { send(it) }
    }
    // 다른 Flow도 동시에 수집하여 채널에 전송
    other.collect { send(it) }
}
```

<br/>


위 예시는 두 개의 플로우를 합치는 과정을 보여준다.

- launch 블록을 사용해 한쪽 Flow를 동시에 수집하고, 수집한 데이터를 send를 통해 채널로 보낸다.
- other.collect는 현재 코루틴에서 진행되며, 결과적으로 두 Flow가 모두 하나의 채널로 합쳐진다.

<br/>


```kotlin
fun <T> contextualFlow(): Flow<T> = channelFlow {
    // IO 컨텍스트에서 어떤 값을 계산하여 전송
    launch(Dispatchers.IO) {
        send(computeIoValue())
    }
    // CPU 컨텍스트에서 다른 값을 계산하여 전송
    launch(Dispatchers.Default) {
        send(computeCpuValue())
    }
}
```

<br/>


이처럼 channelFlow 안에서 다양한 컨텍스트를 병렬적으로 사용할 수 있다.
