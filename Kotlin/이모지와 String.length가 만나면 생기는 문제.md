글자 수 제한을 걸고 싶을 때 보통 

`text.length ≤ x` 

이런 로직을 많이 세운다. 

하지만 이런 로직은 이모지가 들어갈 때, 제대로 돌아가지 않는다. 왜 그럴까?

<br>

## 이모지의 길이가 제대로 측정되지 않는 이유

코틀린이나 자바에서 `String.length`는 문자 수(사람이 보는 글자 수)가 아니라, 내부적으로 **UTF-16 코드 단위**(char) 개수를 뜻한다.

- 문자 ‘가’(U+AC00) 같은 건 UTF-16에서 1개의 char에 들어간다.
- 하지만 대부분의 이모지는 서러게이트(surrogate pair)를 필요로 하거나, 한 이모지가 여러 개의 코드포인트로 합쳐진 “합성 이모지”일 때, UTF-16 상에서 여러 char로 표현될 수 있다.

<br>

즉, 사람이 보기에 ‘한 글자’ 같아도, 기기 내부적으로는 “2~10개 이상의 코드 단위”가 될 수 있다는 점이 문제다.

코틀린에서 `string.length` 대신 `codePointCount`로 코드 포인트 길이를 계산할 수 있다. 하지만 이모지 하나가 여러 코드 포인트로 결합되어 있는 경우에는, 이모지 하나가 여러 개로 세어질 것이기 때문에 충분하지 않다. 

<br>

## 해결 방법

유니코드에서 “시각적으로 하나의 글자(음절이나 합성 이모지 단위)”를 **Extended Grapheme Cluster**(확장 그래페메 군집)라고 한다.

- 즉, “한 글자처럼 보이는 문자 조합을 하나로 묶자”가 유니코드 표준의 개념
- 안드로이드/자바(코틀린)에서는 `BreakIterator` 를 사용해 이 “문자 경계”를 찾을 수 있다.

<br>

`BreakIterator.getCharacterInstance()`

→ 내부적으로 ICU 라이브러리를 이용해서, 각 ‘문자 경계’를 순회할 수 있음.

아래 코드를 보면, BreakIterator로 텍스트를 순회하면서 “그래페메(문자 경계)” 개수를 세고 있다.

<br>

```kotlin
fun String.graphemeLength(): Int {
    if (this.isEmpty()) return 0

    val iterator = BreakIterator.getCharacterInstance()
    iterator.setText(this)

    var count = 0
    var start = iterator.first()

    while (start != BreakIterator.DONE) {
        val end = iterator.next()
        if (end == BreakIterator.DONE) break
        count++
        start = end
    }

    return count
}
```

- next()가 반환할 때마다 한 개의 그래페메(눈에 보이는 글자 단위)를 찾았다고 보고 `count++`
- 결과적으로, 사람 눈에 “한 글자”처럼 보이는 합성 이모지도 count가 1씩 올라가게 된다.

<br>

이렇게 하면 **이모지를 포함한 문자열**에서도, 사람이 시각적으로 보는 대로 글자 수를 제한하거나 표시할 수 있다.

<br>

## +) 복잡도 관점에서 문제가 발생할 수 있지 않나?

`String.length`는 배열의 길이를 바로 리턴하므로 **O(1)**에 가깝지만, `BreakIterator`는 **O(n)** 정도의 시간이 걸린다. 

웬만하면 문제가 되지 않겠지만, 수천~수만 자의 문자열을 입력할 수 있다면 부담이 될 수 있다. 이러한 경우에는 사용자가 입력을 멈췄을 때, 즉 디바운스 기법을 사용해서 최적화를 수행할 수 있을 것 같다. 

또한, n개의 문자열을 처리한다고 해서 임시로 **O(n)** 크기의 추가 배열을 무조건 만들지는 않는다. 따라서 성능 이슈를 너무 걱정하지 않아도 될 것 같다.
