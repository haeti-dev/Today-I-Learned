### mapCathcing

`Result` 타입의 값을 다른 타입으로 안전하게 변환할 때 사용되며, 변환 과정에서 발생할 수 있는 예외를 자동으로 처리해준다.

## 사용 예시

```kotlin
class FetchProductInfoUseCase @Inject constructor(
    private val productRepository: ProductRepository,
) {
    suspend operator fun invoke(productId: String): Result<ProductInfo> {
        val result = productRepository.getProductInfo(productId)

        return result.mapCatching { dto ->
            dto.toProductInfo()
        }
    }
}
```

여기서 `ProductInfoResponseDto`는 다음과 같이 정의되어 있고:

```kotlin
@Serializable
data class ProductInfoResponseDto(
    val id: String,
    val name: String,
    val description: String,
    val price: String,
    val isAvailable: Boolean,
    val status: String,
// ... 기타 필드들
) {
    fun toProductInfo() = ProductInfo(
        id = id,
        isAvailable = isAvailable,
        status = status,
        price = price
    )
}
```

최종적으로 변환되는 `ProductInfo` 모델은 다음과 같다. 

```kotlin
data class ProductInfo(
    val id: String,
    val isAvailable: Boolean,
    val status: String,
    val price: String
)
```

## mapCatching의 장점

1. **안전한 변환**: DTO에서 도메인 모델로 변환하는 과정에서 발생할 수 있는 예외를 자동으로 `Result.failure`로 캡처한다.
2. **체이닝 가능**: Result 타입을 유지하면서 연속적인 변환 작업을 수행할 수 있다.
3. **코드 간결성**: try-catch 블록을 직접 작성하지 않아도 되어 코드가 더 깔끔해진다.

`mapCatching`은 특히 네트워크 응답을 도메인 모델로 변환하는 과정에서 매우 유용하다. 예외 처리를 명시적으로 하지 않아도 되면서도, 안전한 변환을 보장받을 수 있어서 자주 사용할 것 같다.
