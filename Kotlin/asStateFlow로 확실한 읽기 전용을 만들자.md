`asStateFlow()` 내부 구조를 살펴보면 `ReadOnlyStateFlow로` 래핑하는 것을 볼 수 있다. 
- `ReadOnlyStateFlow`는 `CancellableFlow`를 구현하여 “collect 도중에 취소가 발생하면 정상적으로 멈춘다”는 흐름 제어를 가능케 한다.
- 또한, `FusibleFlow`를 통해 “flowOn, buffer 등의 다운스트림 연산자와 최적화(fusion)를 진행”할 수 있다.
- 마지막으로 `fuseStateFlow`, `fuseSharedFlow`, `ChannelFlowOperatorImpl` 등에서 “이미 conflate 중인데 또 conflate가 들어오면 무의미하다”거나 “버퍼/컨텍스트 설정을 추가로 붙여야 한다”는 식의 로직을 구현한다.

이 모든 과정을 통해 `asStateFlow()`는 단순히 “타입만 바꿔주는” 일을 넘어, 코루틴 Flow의 **최적화 및 안전성**을 담보해 주는 구조를 갖추게 된다는 점이 핵심이라고 할 수 있다.

### 관련된 내용 포스팅
[asStateFlow(), 꼭 써야 할까? 해부해보자](https://haeti.palms.blog/as-state-flow-deepdive)
