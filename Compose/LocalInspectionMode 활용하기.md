LocalInspectionMode를 사용하면 Preview에서 어떻게 렌더링되는지 확인할 수 있다. 이런 점을 활용하여 Image를 미리 선언해두면, Preview에서 가짜 이미지를 렌더링하는 로직을 간편하게 사용할 수 있다.

<br>

```kotlin
@Composable
fun UrlImage(
    url: String,
    modifier: Modifier = Modifier,
    contentScale: ContentScale = ContentScale.Fit,
    contentDescription: String? = null,
) {
    if (LocalInspectionMode.current) {
		    // 프리뷰에서 보이는 이미지 
        Image(
            imageVector = ImageVector.vectorResource(R.drawable.img_fake_red),
            contentDescription = contentDescription,
            contentScale = contentScale,
            modifier = modifier
        )
    } else {
		    // 실제 앱에서 보이는 이미지 
		    // 여러가지 커스터마이징 가능
        AsyncImage(
            model = url,
            contentDescription = contentDescription,
            contentScale = contentScale,
            modifier = modifier
        )
    }
}
```

<br>

### 참고

https://developer.android.com/develop/ui/compose/tooling/previews?hl=ko#localinspectionmode

https://naemamdaelo.tistory.com/entry/AsyncImage%EB%A5%BC-Preview%EC%97%90%EC%84%9C-%EB%B3%B4%EB%8A%94-3%EA%B0%80%EC%A7%80-%EB%B0%A9%EB%B2%95
