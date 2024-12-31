Jetpack Compose로 UI를 구축하다 보면 State를 어떻게 관리하느냐가 중요한 화두가 된다. 특히 Compose는 **스냅샷(Snapshot) 기반으로 상태 변경을 추적**하기 때문에, 어떤 기준으로 상태 변화를 ‘같은 값’ 혹은 ‘다른 값’으로 볼 것인지가 꽤 중요한 이슈이다. 여기서 등판하는 개념이 바로 **SnapshotMutationPolicy** 이다.

SnapshotMutationPolicy를 이해하면 다음과 같은 상황에서 좀 더 탄탄한 Compose 코드를 작성할 수 있다.

- 값이 참조만 다를 뿐 실질적으로는 동일할 때, 굳이 리컴포지션(Recomposition)이 일어나지 않게 최적화 가능
- 다중 스냅샷(Multi-Snapshot) 환경에서 상태 충돌이 발생할 때, “이건 충돌인가 아닌가?”를 비교하여 머지(Merge)하는 기준 제공
- 복잡한 데이터 구조를 다룰 때, 값 변경이 어떻게 인지되는지 세밀하게 커스터마이징

자세한 내용 작성 👇

https://haeti.palms.blog/snapshot-mutation-policy
