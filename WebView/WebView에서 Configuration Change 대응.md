안드로이드는 멀티 윈도우 모드를 사용하거나 윈도우를 회전하는 경우 `onDestroy()`가 호출된 후 `onCreate()`가 다시 호출된다. 그럼 웹뷰가 페이지를 다시 로딩하게 되는데 이 현상을 방지하기 위해 `AndroidManifest.xml` 파일에서 `configChanges` 속성을 사용하여 구성 변경 도중 객체를 보존 할 수 있다.

<br/>

```xml
<activity 
					android:name=".MyActivity"
          android:configChanges="orientation|screenSize|screenLayout|keyboardHidden"
          android:label="@string/app_name" />
```

<br/>

이 속성은 화면의 방향성 혹은 크기, 트리거, 키보드 가용성의 변화 등을 감지하여 페이지의 재시작을 막아준다.

항상 정답인 방법은 당연히 아니지만, 정책적으로 대응이 필요할 때 유용하게 사용할 수 있을 것 같다.
