Kotlin의 **Value Class**는 단 하나의 불변 필드만을 가지며, JVM 환경에서는 컴파일 시 내부 값으로 대체되어 불필요한 객체 할당 없이 기본 타입처럼 효율적으로 사용할 수 있다. 이를 통해 값에 의미를 부여하고, 도메인 모델을 보다 안전하게 관리할 수 있다.

<br>

### **Value Class 기본 개념**

1. **단일 불변 필드**
    - Value Class는 오직 하나의 프로퍼티만 보유하여, 값 그 자체의 의미를 명확히 한다.
2. **인라인 최적화**
    - 컴파일러는 value class를 인라인 처리하여 내부 프로퍼티의 값으로 대체함으로써 성능 오버헤드를 줄인다.
3. **타입 안전성**
    - 단순 String이나 Int 대신, 도메인에 특화된 타입을 사용하여 잘못된 값이 전달되는 것을 컴파일 타임이나 객체 생성 시점에 미리 방지할 수 있다.

<br>

### **검증 로직과 팩토리 메서드**

비밀번호를 예시로 들어보자. 아래 코드는 비밀번호의 길이가 최소 8자 이상이어야 함을 생성 시점에 검증한다.
또한, 팩토리 메서드(`from`)를 통해 입력값에 `trim` 같은 전처리 작업을 수행한 후 인스턴스를 생성한다.

<br>

```kotlin
@JvmInline
value class Password(val value: String) {
    init {
        require(value.length >= MIN_LENGTH) { 
            "비밀번호는 최소 $MIN_LENGTH자 이상이어야 합니다. value: $value" 
        }
    }

    companion object {
        private const val MIN_LENGTH = 8
        
        fun from(password: String): Password = Password(password.trim())
    }
}
```

<br>

### 생성자 직접 호출의 문제점

문제는 생성자가 공개되어 있을 경우 발생한다. 예를 들어 아래와 같이 생성자를 직접 호출하면,

```kotlin
val password = Password("123 4567")
```

입력값 "123 4567"은 trim() 처리 없이 그대로 생성자에 전달된다. 이로 인해 공백이 남은 상태로 검증 로직이 실행되고, 의도한 검증(예를 들어 불필요한 공백 제거)을 우회하게 되어 올바르지 않은 값이 생성될 수 있다.

<br>

### 해결책: 생성자를 Private로 만들기

```kotlin
@JvmInline
value class Password private constructor(val value: String) {
    init {
        require(value.length >= MIN_LENGTH) {
            "비밀번호는 최소 $MIN_LENGTH자 이상이어야 합니다. value: $value"
        }
    }

    companion object {
        private const val MIN_LENGTH = 8
        
        // 팩토리 메서드: 외부 입력값을 미리 정제(예, trim)한 후 생성자를 호출함
        fun from(password: String): Password = Password(password.trim())
        
        // invoke 연산자 오버로딩: 생성자 호출처럼 사용할 수 있게 함
        operator fun invoke(password: String): Password = from(password)
    }
}
```

<br>

생성자를 private으로 제한하고 팩토리 메서드(from)를 통해서만 인스턴스를 생성하도록 강제하면, 항상 의도한 전처리와 검증 로직이 적용된다.

또한, invoke 연산자를 오버로딩하면, 사용자는 아래와 같이 일반 생성자 호출 문법을 사용할 수 있다.

<br>

```kotlin
val password = Password(" mySecret12 ")  // 내부적으로 Password.from(" mySecret12 ")가 호출됨
```

<br>

이렇게 하면, 생성자가 private이어도 개발자는 마치 생성자를 직접 호출하는 것처럼 편리하게 사용할 수 있으며, 항상 팩토리 메서드의 전처리 및 검증을 거치게 된다.

<br>

## 성능 비교

같은 구조를 가지는 data class와 value class를 컴파일하면 한 가지 사실을 알 수 있다. value class는 컴파일 시 내부 값을 직접 사용하게 되어 객체를 생성하는 오버헤드를 피할 수 있다. 메모리 사용량이 경우에 따라 다르지만 보통 20% 이상 줄어든다고 하니, 도메인 언어를 명확히 표현하고 싶을 때 이 value class를 사용할 것 같다.
