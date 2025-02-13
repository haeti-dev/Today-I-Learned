### **invokeOnCompletion이란?**

<br>

```kotlin
public fun invokeOnCompletion(handler: CompletionHandler): DisposableHandle
```

`invokeOnCompletion`은 Coroutine Job이 **완료**될 때 호출되는 핸들러를 등록할 수 있는 함수이다. Job이 이미 완료된 상태라면, 핸들러는 즉시 실행되며 Job의 예외 정보(또는 취소 사유, 혹은 null)를 인자로 받는다. 그렇지 않다면 Job이 완료되는 시점에 한 번만 호출된다.

<br>

핸들러가 받는 cause 값은 다음과 같이 해석할 수 있다.

- **null**: Job이 정상적으로 완료됨
- **CancellationException 인스턴스**: Job이  정상적으로 취소된 경우 (오류로 간주하지 않음)
- **그 외의 값**: Job이  실패했을 경우

<br>

또한, invokeOnCompletion은 **동기적으로** 핸들러를 실행하며, 핸들러 내에서 예외가 발생할 경우 `CompletionHandlerException`으로 래핑되어 다시 던져질 수 있으므로, 핸들러 내부에서는 예외가 발생하지 않도록 주의해야 한다.

<br>

[공식 문서](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/invoke-on-completion.html) 참고 

<br>

### **활용 예시**

```kotlin
val sheetState = rememberModalBottomSheetState()
val coroutineScope = rememberCoroutineScope()

ModalBottomSheet(
    modifier = modifier
        .fillMaxWidth()
        .wrapContentHeight()
        .padding(bottom = 16.dp),
    onDismissRequest = {
        onHideBottomSheet()
    },
    sheetState = sheetState,
    dragHandle = null,
) {
    SettingBottomSheetContent(
        onApplyButtonClick = { start, end ->
            coroutineScope.launch {
                sheetState.hide()
            }.invokeOnCompletion {
                // Job이 완료된 후, 시트가 숨겨진 상태이면 추가 작업 수행
                if (!sheetState.isVisible) {
                    onHideBottomSheet()
                }
            }
        },
    )
}
```
<br>

- 사용자가 onApplyButtonClick 이벤트를 발생시키면, `coroutineScope.launch` 내에서 `sheetState.hide()`를 호출해 BottomSheet를 숨긴다.
- `invokeOnCompletion`을 사용하여, `hide()` 작업이 완료된 후에 시트의 상태를 확인한다.
- 만약 BottomSheet가 정상적으로 숨겨졌다면(`!sheetState.isVisible`), 추가적인 작업으로 `onHideBottomSheet()`를 호출합니다.

<br>

이처럼 `invokeOnCompletion`을 활용하면, Coroutine이 종료된 시점에 후속 작업을 안전하게 처리할 수 있어 UI 상태 관리나 비동기 작업의 마무리 처리에 유용하게 사용할 수 있다.
