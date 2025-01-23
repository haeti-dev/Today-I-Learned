![image](https://github.com/user-attachments/assets/4bd0b26a-0bc6-4a86-9e62-bf048d4d631c)


코틀린의 `ifBlank()` 함수는 문자열이 비어있거나 공백으로만 이루어 경우(Blank) 대체값을 람다 형식으로 제공하는 편리한 확장 함수이다. 

<br/>

### AS-IS

```kotlin
val title = if (title.isBlank()) "" else title
```

<br/>

### TO-BE

```kotlin
val title = title.ifBlank { "" }
```

<br/>

위와 같이 간편하게 사용이 가능하며, 람다 함수를 인자로 받을 수 있다. 

if 문을 사용하는게 더 가독성이 좋다고 생각하는 사람이 많을 것이라 생각한다. 코틀린에 능숙한 개발자 집단에서 코드를 짠다면 `ifBlank()`가 더 가독성이 있을 수 있지만, 해당 문법에 친숙하지 않은 사람이 많은 집단에서는 오히려 가독성을 해칠 수 있으니 이 부분은 염두해 두어야 할 것 같다. 

<br/>

또한, null이 아닌 문자열에 대해서만 동작하기에 null 체크가 필요한 경우 safe call 연산자와 함께 사용하면 될 것 같다.
