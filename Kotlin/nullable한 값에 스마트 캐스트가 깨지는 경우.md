## **1. 문제 상황 (Problem Statement)**

- imageRes와 같은 **nullable**한 값을 Compose 함수 내부에서 사용하려고 할 때, `if (imageRes != null)`로 한 번 체크해도 **스마트 캐스트**가 깨지면서 컴파일러가 !! 또는 추가 처리를 요구하는 문제.

```kotlin
if (imageRes != null) {
    Image(
    	imageVector = ImageVector.vectorResource(id = imageRes) <- 에러
      ...
    )
```
<br/>

## **2. 원인 분석**

- **Compose의 비동기적 Recomposition** 특성 때문에, “스마트 캐스트가 완벽히 보장되지 않는다”고 컴파일러가 판단할 수 있음.
- “어딘가에서 imageRes가 바뀌어 null이 될지도 모른다”라고 가정하기 때문.

<br/>

## **3. 해결 방법**

### 1️⃣ **지역 변수로 추출하기**

```kotlin
val resId = imageRes ?: return // null이면 얼리 return

Image(
    imageVector = ImageVector.vectorResource(id = resId),
    ...
)
```

- “null이면 함수 종료” → 이후 코드는 resId를 non-null로 안전하게 사용 가능.
- **스마트 캐스트**가 확실히 적용되어 !! 불필요.

<br/>

### 2️⃣ **?.let { } 블록 사용하기**

```kotlin
imageRes?.let { nonNullRes ->
    Image(
        imageVector = ImageVector.vectorResource(id = nonNullRes),
        ...
    )
}
```

- null이 아닐 때만 블록을 실행하므로 내부에서 nonNullRes는 안전하게 non-null.

<br/>

### 결론

Compose 내에서 nullable 변수를 안전하게 다루기 위해서는 if 조건 뒤에도 스마트 캐스트가 항상 보장되지 않는다는 점을 유의하자. 얼리 return 또는 `?.let` 등으로 가독성을 해치지 않는 선에서 처리하면 좋다.
