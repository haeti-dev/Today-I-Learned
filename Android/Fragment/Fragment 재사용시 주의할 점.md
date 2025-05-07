## **왜 Fragment를 재사용해야 할까?**

안드로이드에서 `replace()`만 써서 Fragment를 교체하면, 화면 회전이나 탭 전환 시마다 새 인스턴스가 생성되어 기존 UI 상태(예: 스크롤 위치, 입력 값 등)가 사라지거나 Fragment가 중복 생성되는 문제가 발생한다.

이로 인해 사용성 저하, 불필요한 메모리 사용, 부자연스러운 화면 전환이 일어나므로, 가능한 한 하나의 Fragment 인스턴스를 재활용하도록 설계해야 한다.

---

## **1. `findFragmentByTag` + `replace` 방식**

```kotlin
private inline fun <reified T : Fragment> replaceFragment() {
    val tag = T::class.java.name
    val fragment = supportFragmentManager
        .findFragmentByTag(tag) as? T
        ?: T::class.java.newInstance()

    supportFragmentManager.commit {
        setReorderingAllowed(true)
        replace(binding.mainFragmentContainer.id, fragment, tag)
    }
}
```

- **장점**
    - 코드가 직관적이고, 기존에 `replace()`로 작성하던 흐름을 크게 바꾸지 않아도 된다.
    - `replace` 호출 시 이전 Fragment가 완전히 제거되므로 메모리 사용량이 일정 수준으로 유지된다.
- **단점**
    - 뷰 계층이 매번 재생성되므로, 스크롤 위치·입력 내용 등 UI 상태가 초기화된다.
    - 애니메이션이나 상태 복원 로직을 별도로 구현해야 할 수 있다.

<br>

## **2. `add` + `show/hide` 방식**

```kotlin
private inline fun <reified T : Fragment> replaceFragment() {
    val tag = T::class.java.name
    val fragment = supportFragmentManager
        .findFragmentByTag(tag) as? T
        ?: T::class.java.newInstance()

    supportFragmentManager.commit {
        setReorderingAllowed(true)
        supportFragmentManager.fragments.forEach { hide(it) }

        if (fragment.isAdded) {
            show(fragment)
        } else {
            add(binding.mainFragmentContainer.id, fragment, tag)
        }
    }
}
```

- **장점**
    - Fragment 뷰가 메모리에 남아 있어 스크롤 위치, 입력 내용, ViewModel 상태 등이 그대로 유지된다.
- **단점**
    - 많은 Fragment를 계속 유지하면 메모리 사용량이 증가할 수 있다.
    - 각 Fragment가 계속 메모리에 머무르므로, 필요 없는 경우 정리(clean up)가 필요하다.

<br>

## 결론

- **상태 보존이 중요할 때** → `add` + `show/hide`
- **백스택·메모리 관리, 단발성 화면 전환** → `replace`
