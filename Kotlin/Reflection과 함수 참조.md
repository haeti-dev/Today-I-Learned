## 1. Reflection이란?

**Reflection**은 런타임 시점에 클래스나 함수, 프로퍼티 등의 메타데이터(Metadata)에 접근하거나, 동적으로 조작하는 기능을 말한다. 다른 언어(Java 등)에서도 흔히 볼 수 있는 기능이지만, 코틀린은 Java의 Reflection에 더해 코틀린만의 문법을 통해 좀 더 간편하게 접근할 수 있게 되어 있다.

JVM은 클래스 로드 시 클래스 로더를 사용해 클래스에 대한 정보를, 즉 Metadata를 Metaspace 영역에 저장한다. 

```kotlin
class Person(val name: String, val age: Int)

fun main() {
    val person = Person("Alice", 20)
    val personKClass = person::class          // (1) 인스턴스를 통한 KClass 얻기
    val personKClass2 = Person::class         // (2) 클래스 참조로 KClass 얻기

    println(personKClass.qualifiedName)       // com.example.Person (패키지 경로 포함)
    println(personKClass2.simpleName)         // Person
}
```

- 클래스의 타입을 얻을 수 있다.

```kotlin
// Person 클래스에 대해
fun main() {
    val person = Person("Bob", 25)
    val nameProperty = Person::name
    val ageProperty = Person::age

    // get/set 같은 Reflection API 사용
    println(nameProperty.get(person))  // "Bob"
    println(ageProperty.get(person))   // 25
}
```

- 특정 클래스의 프로퍼티나 함수를 참조하기 위해 :: 을 사용한다. 이를 통해 프로퍼티 혹은 함수를 메타 데이터로 다룰 수 있다.

이런 Reflection API는 동적으로 타입 정보를 확인해야 할 때나, 어노테이션 처리, 객체 직렬화/역직렬화 라이브러리 등을 구현할 때 활용된다. 다만 **런타임 성능 부담**이 있을 수 있으므로, 불필요하게 남용하는 것은 지양하는 편이 좋다.

<br>

## **2. 코틀린 Reflection의 주요 타입**

1.	**KClass**: 클래스 자체를 나타내는 타입

- 예: Person::class → KClass<Person>

2.	**KProperty**: 프로퍼티를 나타내는 타입

- 예: Person::age → KProperty<Person>

3.	**KMutableProperty**: var처럼 변경 가능한 프로퍼티를 나타내는 타입

4.	**KFunction**: 함수를 나타내는 타입

- 예: ::topLevelFunc → KFunction<R> 형태

이들은 표준 라이브러리에 선언되어 있으며, kotlin.reflect 패키지에 속해 있다.

<br>


## 3. 함수 참조

함수 참조는 람다와 비슷하게 사용되지만, 특정 **이미 선언된 함수**를 **그대로 받아서** 람다처럼 쓸 수 있게 해주는 기능이다. 코틀린에서는 `::함수이름` 또는 `객체::메서드이름`과 같은 문법으로 함수 참조를 생성할 수 있다.

### (1) 기본

```kotlin
fun greet(name: String) {
    println("Hello, $name")
}

fun main() {
    // 함수 참조
    val reference: (String) -> Unit = ::greet

    // 마치 람다처럼 호출할 수 있음
    reference("Kotlin")  // Hello, Kotlin
}
```

- reference라는 변수가 `::greet`을 참조하게 되었고, 이후에는 `reference("Kotlin")`처럼 람다 호출하듯 사용할 수 있다.

### (2) 맴버 함수 참조

```kotlin
class Printer {
    fun printMessage(msg: String) {
        println("Printer prints: $msg")
    }
}

fun main() {
    val printer = Printer()
    val boundRef: (String) -> Unit = printer::printMessage
    boundRef("Hello")  // Printer prints: Hello
}
```

- 이미 특정 객체가 있을 때 그 객체의 멤버 함수로 바로 참조한다.

```kotlin
val unboundRef: (Printer, String) -> Unit = Printer::printMessage
unboundRef(Printer(), "Hi") // Printer prints: Hi
```

- 아직 인스턴스가 없고, 클래스의 함수 시그니처만 참조할 경우

### (3) 생성자 참조

```kotlin
class Person(val name: String, val age: Int)

fun main() {
    val createPerson: (String, Int) -> Person = ::Person
    val alice = createPerson("Alice", 20)

    println(alice.name) // Alice
    println(alice.age)  // 20
}
```

- 생성자도 함수처럼 참조하여 사용할 수 있다.

### (4) 프로퍼티 참조

```kotlin
val nameRef: (Person) -> String = Person::name
val person = Person("Eve", 30)
println(nameRef(person)) // "Eve"
```

- get 참조

```kotlin
var Person.nickname: String
    get() = ...
    set(value) { ... }

val setNickname: (Person, String) -> Unit = Person::nickname.set
setNickname(person, "Genie")
```

- set 참조

  <br>


## **4. Reflection과 함수 참조의 차이점**

### **Reflection (KFunction, KProperty 등)**

- 메타데이터에 직접 접근하여, KFunction.call(), KProperty.get(), KProperty.set() 등의 **Reflection API**를 사용할 수 있다.
- 클래스의 전체 멤버 목록을 얻고, 동적으로 어떤 함수를 호출하거나 값을 설정해야 할 때 활용된다.
- 상대적으로 런타임 부하가 크며, Proguard나 난독화 환경에서 주의가 필요하다.

### **함수 참조**

- 이미 선언된 함수를 “람다처럼” 핸들링하는 간단한 기능이다.
- Reflection API를 직접 다루지 않으며, 함수의 시그니처에 맞춰서 쓰기만 하면 된다.
- 주로 고차 함수(High-order function)나 함수형 API 쓸 때 가독성과 재사용성을 높이는 용도로 사용된다.

<br>


## 참고자료

https://everyday-develop-myself.tistory.com/332
