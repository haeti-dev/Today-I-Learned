React에서는 복잡한 UI 로직을 조합 방식으로 풀어내고, 과도한 Props Drilling을 Context API를 사용하여 제거하였다. 이런 패턴을 Compound Component 패턴이라고 지칭하며, Jetpack Compose에서는 이 패턴을 Context API 대신 람다 리시버로 구현할 수 있다. 

다이얼로그나 바텀 시트와 같이 UI 로직이 조건부 상황에 많이 놓여있고, Slot API로는 간단하게 해결하지 못하는 경우 가장 유용하게 사용된다. 

### 작성한 블로그

https://haeti.palms.blog/compose-compound-components
