## 1. ConnectivityManager 핵심 개념

### ConnectivityManager

> 기기 내 모든 네트워크 연결 상태에 대한 쿼리에 응답하는 클래스
> 

- `activeNetwork` : 현재 “기본(default)” 네트워크
- `getNetworkCapabilities(network)` : 해당 네트워크의 속성(인터넷 유효성·비계량·셀룰러/와이파이 등)
- `registerNetworkCallback(request, callback)` : NetworkRequest에 부합하는 네트워크가 추가/소실/변경될 때 콜백

### NetworkRequest

> 모니터링할 네트워크의 ‘조건’을 정의
> 

- `.addCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET)` : 인터넷 사용 가능
- `.addCapability(NetworkCapabilities.NET_CAPABILITY_VALIDATED)` : 실제 외부와 통신 가능한 상태
- (옵션) .addTransportType(...) : WIFI, CELLULAR 등 특정 전송 수단 지정

### NetworkCallback

> 네트워크 변화 시점에 호출되는 콜백
> 

- `onAvailable(network)` : 조건을 만족하는 네트워크가 연결됐을 때
- `onLost(network)` : 네트워크 연결이 끊겼을 때
- `onLosing(network, maxMsToLive)` : 곧 연결성이 약화될 때
- `onCapabilitiesChanged(network, caps)` : 네트워크 속성이 바뀔 때 (예: Wi-Fi→셀룰러)

### 필요권한

- `android.permission.ACCESS_NETWORK_STATE` : 애플리케이션에서 네트워크에 관한 정보에 접근하기 위해 필요함

<br>

## 2. 사용 예시

```kotlin
class NetworkStatusTracker @Inject constructor(
    private val connectivityManager: ConnectivityManager,
    @IoDispatcher private val ioDispatcher: CoroutineDispatcher
) {
    // 현재 등록된 네트워크를 보관
    private val activeNetworks = mutableSetOf<Network>()

    @SuppressLint("MissingPermission")
    val isConnected: Flow<Boolean> = callbackFlow {
        // 1) 콜백 정의
        val callback = object : ConnectivityManager.NetworkCallback() {
            override fun onAvailable(network: Network) {
                activeNetworks.add(network)
                trySend(checkIfConnected())
            }
            override fun onLost(network: Network) {
                activeNetworks.remove(network)
                trySend(checkIfConnected())
            }
        }

        // 2) “인터넷+검증된(validated)” 네트워크 요청
        val request = NetworkRequest.Builder()
            .addCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET)
            .addCapability(NetworkCapabilities.NET_CAPABILITY_VALIDATED)
            .build()

        // 3) 초기 상태 발행
        connectivityManager.activeNetwork?.let { activeNetworks.add(it) }
        trySend(checkIfConnected())

        // 4) 콜백 등록/해제
        connectivityManager.registerNetworkCallback(request, callback)
        awaitClose { connectivityManager.unregisterNetworkCallback(callback) }
    }
    
    // 현재 activeNetworks 중 하나라도 VALIDATED 상태이면 true
    private fun checkIfConnected(): Boolean =
        activeNetworks.any { net ->
            connectivityManager.getNetworkCapabilities(net)
                ?.hasCapability(NetworkCapabilities.NET_CAPABILITY_VALIDATED) == true
        }.also { /* 추가로 로깅 가능 */ }
      .let { result -> result } 
      .flowOn(ioDispatcher) // I/O 스레드에서 처리
}
```

<br>

위 방식의 단점

1. **약한 신호(품질 저하) 탐지 불가**
    - onLosing·onCapabilitiesChanged 미구현
2. **콜백 중복 등록**
    - Flow를 구독할 때마다 registerNetworkCallback 호출
3. **구독자마다 새로운 흐름**
    - 다수 구독 시 리소스 낭비

<br>

## 3. 최적의 활용 방안

### 1) 싱글톤 → DI기반 싱글톤

- 앱 내에서 네트워크 콜백을 한 번만 등록해 중복 호출·리소스 낭비 방지
- 애플리케이션 전체에서 동일한 StateFlow/SharedFlow를 재사용
- 생명주기 관리의 명확성
- 런타임 에러 방지

```kotlin
@Singleton
class DefaultConnectivityObserver @Inject constructor(
  private val cm: ConnectivityManager,
  @IoDispatcher private val dispatcher: CoroutineDispatcher,
  @ApplicationScope private val appScope: CoroutineScope
) : ConnectivityObserver {
  override val status: StateFlow<Status> = callbackFlow<Status> {
    // ...NetworkCallback 등록 로직...
  }
  .flowOn(dispatcher)
  .stateIn(
    scope = appScope,
    started = SharingStarted.Lazily,   // 구독 시 한 번만 콜백 등록
    initialValue = Status.Unavailable
  )
}
```

- Stopped 상태에서 다시 재시작되는 시점도 고려해 `SharingStarted.WhileSubscribed(...)` 옵션도 검토해볼만 함

<br>

### 2) Validated + Transport 구분

- 단순 “연결됨” 여부가 아니라 실제 인터넷 접근 가능(VALIDATED) 상태를 보장

```kotlin
val request = NetworkRequest.Builder()
  .addCapability(NET_CAPABILITY_INTERNET)
  .addCapability(NET_CAPABILITY_VALIDATED)
  .addTransportType(TRANSPORT_WIFI)
  .addTransportType(TRANSPORT_CELLULAR)
  .build()
```

- 필요에 따라 addTransportType(TRANSPORT_WIFI)과 같이 전송 매체별 세분화 정책을 반영할 수 있음

<br>

### **3) 완전한 NetworkCallback 구현**

- 품질 저하 전조(약화) 이벤트까지 캐치해 사용자 경험(UX) 최적화
- 속성 변경(e.g. Wi-Fi→셀룰러)에도 즉각 대응

```kotlin
val callback = object : ConnectivityManager.NetworkCallback() {
  override fun onAvailable(network: Network) { /* 상태 = Available */ }
  override fun onLosing(network: Network, maxMsToLive: Int) { /* 상태 = Losing */ }
  override fun onLost(network: Network) { /* 상태 = Unavailable */ }
  override fun onCapabilitiesChanged(network: Network, caps: NetworkCapabilities) {
    /* VALIDATED 여부 재검사 */
  }
}
```

- `onAvailable(network)` : 네트워크 확보
- `onLosing(network, maxMsToLive)` : 곧 끊길 예정 → 리소스 예비 처리, 로컬 캐시 활용
- `onLost(network)` : 완전 단절
- `onCapabilitiesChanged(network, caps)` : 속성 변경

<br>

### **4) 공유 가능한 “Hot” Flow 만들기**

- cold callbackFlow가 구독자마다 독립적으로 콜백을 등록하지 않도록
- 앱 전역에서 “실시간”으로 동일한 스트림을 구독

```kotlin
val statusFlow = callbackFlow<Status> {
  // 등록 로직
}.shareIn(
  scope = appScope,
  started = SharingStarted.WhileSubscribed(stopTimeoutMs = 5_000),
  replay = 1
)
```

- `stateIn` 또는 `shareIn` 사용
- scope는 ApplicationScope나 SupervisorScope 등 앱 생명주기 범위로
- `replay = 1` 로 초기값/최신값 보장
- `stopTimeoutMs` 로 마지막 구독 해제 후 콜백 자동 해제 시점 조절

<br>

### 5) 백그라운드 전략

- 네트워크 제약에 따라 백그라운드 작업 자동 실행
- 연결 끊김 시 로컬 캐시에 저장, 복구 시점에 동기화

```kotlin
val work = OneTimeWorkRequestBuilder<SyncWorker>()
  .setConstraints(
    Constraints.Builder()
      .setRequiredNetworkType(NetworkType.CONNECTED)
      .build()
  )
  .build()
WorkManager.getInstance(context).enqueue(work)
```

1. 네트워크 없음 → 로컬 DB(Room), DataStore에 쓰기
2. 연결 복구(onAvailable/VALIDATED) 시점에 **일괄** 동기화 트리거
3. 실패 시 지수 백오프 재시도

<br>

### 6) UI / ViewModel에서의 활용

- 네트워크 상태 변화에 맞춰 UI(오프라인 배너, retry 버튼 등) 자동 제어
- ViewModelScope 내에서 Flow 연산자로 간결하게 구현

```kotlin
class MainViewModel @Inject constructor(
  connectivityObserver: ConnectivityObserver
): ViewModel() {
  val networkStatus = connectivityObserver.status
    .distinctUntilChanged()
    .onEach { status ->
      when (status) {
        Status.Available   -> _uiState.update { it.hideOfflineBanner() }
        Status.Losing     -> _uiState.update { it.showLowQualityWarning() }
        Status.Unavailable-> _uiState.update { it.showOfflineBanner() }
      }
    }
    .launchIn(viewModelScope)
}
```

- `distinctUntilChanged()` : 중복 알림 방지
- `debounce(500)` : 연속 변화 시 과도한 UI 업데이트 제어
- `filter { it == Status.Unavailable }` : 오직 오프라인 상태만 별도 로직

요약하자면

1. **한 번만 등록**된
2. **정확히 필터링**된
3. **상태를 세분화**해 감지하며
4. **앱 전역**으로 공유되는
5. **백그라운드 작업과 연동**되는
6. **UI에서 간결하게** 활용할 수 있는

네트워크 상태 감시 구조를 만드는 것이 좋다

<br>

---

<br>

## 참고 자료

https://developer.android.com/training/monitoring-device-state/connectivity-status-type?hl=ko&_gl=1*wce7bp*_up*MQ
