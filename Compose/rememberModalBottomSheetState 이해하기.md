`rememberModalBottomSheetState()`는 Jetpack Compose의 Material3에서 제공하는 BottomSheet의 상태를 관리하기 위한 Composable 함수이다. 이 함수는 BottomSheet의 다양한 상태(확장, 축소, 부분 확장 등)를 제어하고 관리하는 데 사용된다.

<br>

## 기본 구조

```kotlin
@Composable
@ExperimentalMaterial3Api
fun rememberModalBottomSheetState(
    skipPartiallyExpanded: Boolean = false,
    confirmValueChange: (SheetValue) -> Boolean = { true },
) = rememberSheetState(skipPartiallyExpanded, confirmValueChange, Hidden)
```

<br>

## 주요 파라미터

### 1. skipPartiallyExpanded

- **기본값**: `false`
- **역할**: BottomSheet의 부분 확장 상태를 건너뛸지 여부를 결정
- **동작 방식**:
    - `true`: BottomSheet는 항상 완전히 확장된 상태(Expanded)와 숨김 상태(Hidden) 사이에서만 전환된다.
    - `false`: BottomSheet가 충분히 큰 경우, 부분 확장 상태를 지원한다.
    
<br>

### 2. confirmValueChange

- **기본값**: `{ true }`
- **역할**: 상태 변경을 확인하거나 거부할 수 있는 콜백 함수
- **파라미터**: `SheetValue` - 변경하려는 상태 값
- **반환값**: Boolean (상태 변경 허용 여부)
- **사용 예시**:
    
    ```kotlin
    val sheetState = rememberModalBottomSheetState(
        confirmValueChange = { newState ->
            // 특정 조건에서만 상태 변경을 허용
            newState != SheetValue.Hidden
        }
    }
    ```

    <br>

## SheetValue 상태

BottomSheet는 다음 세 가지 상태를 가질 수 있다

- `Hidden`: 숨겨진 상태
- `PartiallyExpanded`: 부분적으로 확장된 상태 (skipPartiallyExpanded가 false일 때만 가능)
- `Expanded`: 완전히 확장된 상태

<br>

## 주요 사용법

```kotlin
val sheetState = rememberModalBottomSheetState(skipPartiallyExpanded = true)
val coroutineScope = rememberCoroutineScope()

// 상태 제어 예시
LaunchedEffect(key1) {
    sheetState.show() // 보여주기
    sheetState.hide() // 숨기기
}

// 현재 상태 확인
LaunchedEffect(sheetState.currentValue) {
    when (sheetState.currentValue) {
        SheetValue.Hidden -> { /* 처리 */ }
        SheetValue.Expanded -> { /* 처리 */ }
        SheetValue.PartiallyExpanded -> { /* 처리 */ }
    }
}
```

<br>

## 주의할 점

1. **상태 관리**
    - BottomSheet의 상태는 가능한 상위 컴포넌트에서 관리하는 것이 좋다.
    - 상태 변경은 coroutineScope 내에서 수행해야 한다.
2. **성능 최적화**
    - `skipPartiallyExpanded`를 `true`로 설정하면 중간 상태가 없어져 더 부드러운 UX를 제공할 수 있다.
    - 불필요한 리컴포지션을 피하기 위해 상태 변경 로직을 `LaunchedEffect` 내에서 처리한다.
3. **예외 처리**
    - `confirmValueChange`를 활용하여 특정 상황에서의 상태 변경을 제한할 수 있다.
    - 상태 변경 실패에 대한 적절한 처리가 필요하다.
