# **Jetpack Compose의 TextMeasurer API**

## **1. TextMeasurer란?**

- Jetpack Compose에서 텍스트 크기를 측정하는 API.
- 측정된 결과를 기억하고 캐싱하여 불필요한 연산을 방지.


<br>

### **2. TextMeasurer의 기억 방식**

- `rememberTextMeasurer()`를 사용하여 상태를 기억.
- 기본적으로 **8개의 캐시 슬롯**(DefaultCacheSize: 8)을 사용.
- TextMeasurer 객체가 생성되면 measure() 함수를 통해 텍스트 크기를 측정 가능.

```kotlin
val textMeasurer = rememberTextMeasurer()
```

```kotlin
private val textLayoutCache: TextLayoutCache? = if (cacheSize > 0) {
    TextLayoutCache(cacheSize)
} else null
```

- 캐싱을 원하지 않는다면 cacheSize = 0으로 설정하면 TextLayoutCache를 사용하지 않음.

<br>

### **3. measure() 함수의 동작 방식**

- 텍스트 크기 측정을 수행하며 내부적으로 캐싱을 활용.
- 캐시 여부는 `skipCache`와 `textLayoutCache` 변수를 통해 결정.

```kotlin
val cacheResult = if (!skipCache && textLayoutCache != null) {
  textLayoutCache.get(requestedTextLayoutInput)
} else null
```

- 초기 호출 시 `skipCache`는 `false`이며 `textLayoutCache`가 `null`이므로 새로운 측정 객체를 생성.
- 이후 호출에서는 캐싱된 값을 복사하여 반환.

```kotlin
return if (cacheResult != null) cacheResult.copy(
  layoutInput = requestedTextLayoutInput,
  size = constraints.constrain(
      IntSize(
          cacheResult.multiParagraph.width.ceilToInt(),
          cacheResult.multiParagraph.height.ceilToInt()
      )
  )
)
```

- **LRU(Least Recently Used) 알고리즘**을 사용하여 캐시 관리.

<br>

### **4. 리컴포지션 발생 시 불필요한 measure() 호출 문제**

- 리컴포지션이 발생하면 measure()가 여러 번 호출될 수 있음.
- 캐싱 덕분에 새로운 객체를 만들지 않지만, **리컴포지션을 최소화하는 것이 더욱 효율적**.

<br>

### **5. measure() 호출 최소화 방법**

- remember를 활용하여 measure() 결과를 저장.

```kotlin
val textLayoutResult = remember(Unit) {
  textMeasurer.measure(
    text = "테스트",
    style = TextStyle(fontSize = 12.sp),
  )
}
```

- 이렇게 하면 **리컴포지션이 발생해도 measure()가 다시 호출되지 않음**.

<br>

### **6. drawWithCache와의 관계**

- **drawWithCache는 measure() 호출을 최적화하는 방법 중 하나**.
- measure()는 성능 비용이 크기 때문에 **그리기 영역이 변경될 때만 측정하도록 배치**.

```kotlin
val textMeasurer = rememberTextMeasurer()

Spacer(
    modifier = Modifier
        .drawWithCache {
            val measuredText =
                textMeasurer.measure(
                    AnnotatedString(longTextSample),
                    constraints = Constraints.fixedWidth((size.width * 2f / 3f).toInt()),
                    style = TextStyle(fontSize = 18.sp)
                )

            onDrawBehind {
                drawRect(pinkColor, size = measuredText.size.toSize())
                drawText(measuredText)
            }
        }
        .fillMaxSize()
)
```

- drawWithCache 내부에서 measure()를 호출하면, **그리기 영역이 변경될 때만 측정**되므로 성능이 향상됨.

<br>

### **🔹 핵심 요약**

1. **TextMeasurer는 Jetpack Compose에서 텍스트 크기 측정을 위한 API**로, rememberTextMeasurer()를 사용해 상태를 유지.
2. 기본적으로 **8개의 캐시 슬롯을 사용하여 측정 값을 저장**하고, LRU 알고리즘을 통해 캐시를 관리.
3. **measure()는 캐싱을 활용하지만, 리컴포지션이 발생하면 계속 호출될 가능성이 있음**.
4. 이를 방지하기 위해 **remember(Unit)을 사용하여 measure()의 결과를 저장**하면 불필요한 호출을 줄일 수 있음.
5. drawWithCache를 사용하면 **그리기 영역이 변경될 때만 measure()가 호출되도록 최적화 가능**.

이렇게 하면 **불필요한 연산을 줄이고, 성능을 최적화**할 수 있음
