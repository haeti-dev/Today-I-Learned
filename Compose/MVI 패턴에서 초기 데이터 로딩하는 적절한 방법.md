## 기존 접근법의 문제점 분석

### 1. ViewModel에서 init 블록 사용

```kotlin
@HiltViewModel
class ForYouViewModel @Inject constructor(
    private val getTopicsUseCase: GetTopicsUseCase,
    private val getNewsUseCase: GetNewsUseCase
) : BaseViewModel<ForYouState, ForYouEvent, ForYouEffect>(
    initialState = ForYouState.initial(),
    reducer = ForYouScreenReducer()
) {
    init {
        // Get data via UseCases
    }
    
    fun onTopicClick(topicId: String) {
        sendEffect(
            effect = ForYouEffect.NavigateToTopic(
                topicId = topicId
            )
        )
    }
}
```

- **장점** : Configuration Change시 재로딩 방지
- **단점**
    - 데이터가 로드되는 시기를 제어할 수 없음
    - 테스트 환경에서 중간에 로직을 삽입할 수 없다.

<br>

### 2. LaunchedEffect 사용

```kotlin
@Composable
fun MyScreen(viewModel: MyViewModel) {
    LaunchedEffect(Unit) {
        viewModel.loadData() // 재구성 시마다 호출될 수 있음
    }
}
```

- **장점** : 명시적으로 로딩을 트리거 할 수 있다.
- **단점**
    - Configuration Change 시 데이터가 재로딩된다.
    - Configuration Change보다 오래 지속되는 뷰모델의 목적을 무효화한다.

<br>


## 해결 방법

⇒ Flow의 확장 함수 `onStart` + `stateIn`

<br>


`onStart`

> “*Returns a flow that invokes the given action **before** this flow starts to be collected.”*
> 
- `onStart`는 Flow 컬렉션이 시작되기 전에 특정 동작을 수행할 수 있게 해준다.
- 예를 들어, 초기 데이터를 로딩하는 함수를 호출하여 화면을 관찰하는 시점에서 데이터를 불러오도록 할 수 있다.

<br>


`stateIn`

- `stateIn`은 Flow를 `StateFlow`로 변환해 현재 상태를 계속 보존하면서 관찰할 수 있도록 한다.
- 이 방식은 상태를 캐싱하여 UI가 언제든지 최신 상태를 구독할 수 있게 해준다.
- `stateIn`을 사용하면 여러 구독자가 있을 때 동일한 데이터를 공유할 수 있고, 마지막 구독자가 사라지면 일정 시간 후 자동으로 컬렉션을 중단할 수 있다. 예를 들어, 아래와 같이 `viewModelScope`와 `SharingStarted` 옵션을 지정하여 구독자가 있을 때만 데이터 로딩을 활성화할 수 있다
    
    ```kotlin
    val state: StateFlow<S> by lazy {
        _state
            .onStart { initialDataLoad() }
            .stateIn(
                scope = viewModelScope,
                started = SharingStarted.WhileSubscribed(5000),
                initialValue = initialState
            )
    }
    ```
    
- 변환 시 초기값(`initialValue`)을 지정함으로써, 상태가 곧바로 사용될 수 있도록 보장한다. 이는 UI 계층에서 데이터 로드를 기다리는 동안에도 안정적인 초기 상태를 제공하는 데 유리하다.

<br>

### 공통 초기 데이터 로드 로직을 갖춘 BaseViewModel 구현

```kotlin
abstract class BaseViewModel<S>(initialState: S) : ViewModel() {
    private val _state = MutableStateFlow(initialState)
    val state: StateFlow<S> by lazy {
        _state
            .onStart { initialDataLoad() }
            .stateIn(
                scope = viewModelScope,
                started = SharingStarted.WhileSubscribed(5000),
                initialValue = initialState
            )
    }

    open fun initialDataLoad() {}
}
```

<br>

### 오버라이딩하여 초기화 로직 구현

```kotlin
class ProductViewModel(
    private val loadProducts: LoadProductsUseCase
) : BaseViewModel<ProductState>(ProductState.Loading) {

    override fun initialDataLoad() {
        viewModelScope.launch {
            sendEvent(ProductEvent.Loading)
            loadProducts().collect { result ->
                _state.update { result.toState() }
            }
        }
    }

```

<br>


### 특징

- ANR을 방지하기 위해 5초의 유휴시간을 가진다.
- 첫 구독자 발생 시 데이터를 로딩한다.
- Configuration Change 시 재로딩이 발생하지 않는다.
- 모든 ViewModel에 동일한 패턴을 적용할 수 있다.
- 대규모 데이터 셋을 로딩할 때는 `SharingStarted.Lazily` 모드를 고려해야 한다.
- 초기 데이터 캐싱 시 오프라인 지원을 고려하면 좋을 것 같다.

<br>


## 최종 비교

| **접근 방식** | **특징** | **설명** |
| --- | --- | --- |
| **ViewModel init 블록** | 1. ViewModel이 생성될 때마다 loadData가 호출됨 2. 테스트 시 로딩 시점 제어 불가 | 초기 데이터 로딩 시점을 세밀하게 제어하기 어렵고, 불필요한 네트워크 호출 등 부작용 발생 가능 |
| **LaunchedEffect in Composable** | 1. Composable이 재구성될 때마다 loadData가 재실행될 위험 2. UI와 ViewModel 생명주기 분리 문제 발생 | UI 재구성 시 중복 데이터 로딩 가능, 상태 불일치 등의 문제가 발생할 수 있음 |
| **onStart + stateIn** | 1. 컬렉션 시작 시 정확히 한 번만 초기 데이터 로딩 실행 2. 최초 구독자 발생 시 데이터 로딩 트리거 | Flow의 콜렉션 시점에 맞춰 데이터 로딩을 실행하고, 상태를 캐싱하여 안정적인 공유 및 제어 가능 |

<br>


## 출처

https://proandroiddev.com/loading-initial-data-properly-with-mvi-5e54edd8ae56
