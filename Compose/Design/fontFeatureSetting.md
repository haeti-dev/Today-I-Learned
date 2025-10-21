서체(font)에 따라 숫자의 폭이 서로 다를 수 있다.

예를 들어 **비등폭(non-tabular) 숫자**를 사용하는 폰트의 경우,

1, 2, 0 등의 폭이 제각각이라 "1234"와 "0000"의 문자열 길이가 다르게 렌더링된다.

이는 숫자가 정렬되는 UI(예: 타이머, 금액 표시 등)에서 시각적으로 어색함을 유발한다.

### **💡 해결 방법**

fontFeatureSettings 속성에 **탭형 숫자(tabular numbers)** 기능을 적용하면

모든 숫자가 동일한 폭으로 렌더링되어 폭 불일치 문제를 해결할 수 있다.

```kotlin
Text(
    text = "1234",
    style = MaterialTheme.typography.h4.copy(
        fontFeatureSettings = "tnum", // tabular numbers
    ),
)
```

### **🧩 fontFeatureSettings 주요 옵션들**

fontFeatureSettings는 OpenType 폰트 기능을 직접 제어할 수 있는 속성이다.

대표적으로 아래와 같은 설정을 사용할 수 있다.

| **기능** | **설정 문자열** | **설명** |
| --- | --- | --- |
| Tabular Numbers | "tnum" | 숫자 폭을 동일하게(등폭) 렌더링 |
| Proportional Numbers | "pnum" | 숫자 폭을 자연스럽게 가변 폭으로 렌더링 |
| Oldstyle Figures | "onum" | 숫자를 본문용 소문자 스타일로 렌더링 |
| Lining Figures | "lnum" | 숫자를 대문자 높이로 렌더링 |
| Small Caps | "smcp" | 소문자를 소문자 크기의 대문자 스타일로 표시 |
| Fractions | "frac" | 분수를 예쁘게 표현 (예: 1/2 → ½) |

여러 기능을 동시에 적용할 때는 쉼표로 구분하여 설정할 수 있다.

```kotlin
fontFeatureSettings = "tnum, lnum"
```
