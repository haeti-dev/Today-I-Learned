안드로이드에서 문자열 리소스를 정의할 때 `translatable` 속성을 사용하여 해당 문자열의 번역 여부를 제어할 수 있다.

```xml
<string name="yes" translatable="false">네</string>
<string name="no" translatable="false">아니요</string>
```

`translatable="false"` 로 설정된 문자열은

- Translations Editor에서 번역 대상으로 표시되지 않는다.

  ![image](https://github.com/user-attachments/assets/1bc6db5a-919b-45cb-b11e-899f3630b8b9)

- 다른 언어 리소스 파일에서 해당 문자열을 정의해도 무시된다.
- 앱은 항상 기본 strings.xml의 값을 사용한다.

<br/>

![image](https://github.com/user-attachments/assets/704bb107-cf42-4c8d-b782-0983ee6b981c)


언어별로 리소스 파일을 구성할 때는

- values/string.xml
- values-en/string.xml
- values-fr/string.xml

와 같이 특정 언어에 대한 번역을 수행할 수 있다. 특정 언어에 대한 번역이 없는 경우 시스템은 자동으로 기본 strings.xml의 값을 사용한다.

<br/>

## 느낀 점

- 원래 strings.xml 파일로 문자열을 추출하면 자동으로 번역되는 줄 알았는데, 아니였다.
- 브랜드명, 고유명사, 기술적인 용어 등은 `translatable=”false”`을 통해 번역을 의도적으로 수행하지 않는 것이 좋아 보인다.
- 글로벌 유저가 많은 앱인 경우 디폴트 언어를 영어로 하는 것이 좋은 전략이 될 수도 있을 것 같다.
- Translation Editor에 주석을 기준으로 폴더링이라던지, 카테고리화를 할 수 있는 GUI를 추가로 지원하면 더더욱 사용하기 좋을 것 같다.
- 매번 배포할 때마다 strings.xml 파일에 추가되는 문자열들에 대해 수동으로 번역을 수행하는 것은 리소스가 많이 드니, 외부 번역 API(파파고, DeepL 등)을 활용하여 파일 경로를 통해 자동으로 번역을 수행하면 리소스를 효율적으로 사용할 수 있을 것 같다.

### 레퍼런스

https://developer.android.com/studio/write/translations-editor?hl=ko