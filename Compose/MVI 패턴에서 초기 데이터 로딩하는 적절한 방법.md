## 기존 접근법의 문제점 분석

## 1. ViewModel에서 init 블록 사용

- **장점** : Configuration Change시 재로딩 방지
- **단점**
    - 데이터가 로드되는 시기를 제어할 수 없음
    - 테스트 환경에서 중간에 로직을 삽입할 수 있다.
 
<br>

### 2. LaunchedEffect 사용

- **장점** : 명시적으로 로딩을 트리거 할 수 있다.
- **단점**
    - Configuration Change 시 데이터가 재로딩된다.
    - Configuration Change보다 오래 지속되는 뷰모델의 목적을 무효화한다.

<br>

## 해결 방법

Flow의 확장 함수 `onStart` + `stateIn`

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

**사용처**

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
}
```

- ANR을 방지하기 위해 5초의 유휴시간을 가진다.
- 첫 구독자 발생 시 데이터를 로딩한다.
- Configuration Change 시 재로딩이 발생하지 않는다.
- 모든 ViewModel에 동일한 패턴을 적용할 수 있다.
- 대규모 데이터 셋을 로딩할 때는 `SharingStarted.Lazily` 모드를 고려해야 한다.
- 초기 데이터 캐싱 시 오프라인 지원을 고려하면 좋을 것 같다.

<br>

## 출처

https://proandroiddev.com/loading-initial-data-properly-with-mvi-5e54edd8ae56
