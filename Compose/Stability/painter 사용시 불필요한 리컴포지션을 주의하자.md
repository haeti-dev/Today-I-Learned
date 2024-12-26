Compose에서 Image 사용시 여러 옵션이 있다. 그 중 Painter를 사용해 아이콘에 이미지를 렌더링할 경우 painterResource를 사용한다. 내부적으로 @Stable이나, @Immutable 어노테이션을 사용하지 않을 뿐더러 원시타입도 아니기 때문에, 파라미터로 painter를 받으면 불필요한 리컴포지션이 발생할 수 있다.

이러한 문제상황을 해결하기 위해 두 가지 옵션이 있다.
1. 내부적으로 @Immutable 어노테이션을 사용하는 ImageVector를 사용해 이미지를 렌더링하는 것.
2. 외부에서 @DrawbleRes를 사용하거나 하지 않은 Int 타입의 Drawable ID를 받아서 painterResource로 painter를 생성한뒤, Image에 적용하는 것  


### 참고
https://tech.wonderwall.kr/articles/recomposition/
https://developer.android.com/develop/ui/compose/graphics/images/optimization?hl=ko#pass-url
