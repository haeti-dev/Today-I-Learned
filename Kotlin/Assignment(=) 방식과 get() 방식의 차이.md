## 1. Assignment(=) 방식

```kotlin
class Test1 {
    private var _word = "test"
    val word: String = _word
}
```

바이트 코드를 자바 코드로 디컴파일을 수행하면 아래와 같이 변환된다.

```java
public final class Test1 {
   private String _word = "test";
   @NotNull
   private final String word;

   @NotNull
   public final String getWord() {
      return this.word;
   }

   public Test1() {
      this.word = this._word;
   }
}
```

<br>

정리하자면, 

1. `word`라는 **final 필드**가 하나 더 생김.
2. 생성자에서 `word = _word`로 초기화가 이루어짐.
3. `getWord()`는 이 `final word` 값을 반환.
4. `_word`를 변경하더라도, **이미 할당된 final word는 변하지 않음**.

<br>

따라서 **초기화 시점**에 _word의 값을 **복사**해서 word가 결정되며, **이후 _word가 변해도 word는 바뀌지 않는다.**

<br>

## 2. get() 방식

```java
class Test1 {
    private var _word = "test"
    val word: String
        get() = _word
}
```

자바 코드로 디컴파일을 시도하면, 

```java
public final class Test1 {
   private String _word = "test";

   @NotNull
   public final String getWord() {
      return this._word;
   }
}
```
<br>

- `word`라는 **별도의 필드**는 존재하지 않음.
- `getWord()` 메서드는 호출될 때마다 _word를 바로 반환.
- `_word` 값이 바뀌면 `getWord()`로 조회했을 때도 **그 변경 사항이 그대로 반영됨.**

<br>

## 결론

- Assignment 방식은 객체 생성 시점에 값을 복사하고 그 이후에 변경되지 않는 반면, get() 방식은 get() 메서드가 호출될 때마다 변경된 값을 갖게된다. **따라서 변화를 항상 반영해야 하는 상황에서는 get() 방식을 사용해야 한다. 상수처럼 사용할 때는 Assignment 방식을 써도 무방하다.**
- 엄청 큰 차이는 아니지만, Assignment 방식은 final 프로퍼티를 하나 더 가지므로 미미한 성능 차이가 있을 수 있다.
