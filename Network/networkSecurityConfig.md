`android:usesCleartextTraffic="true"` 속성은 앱이 암호화되지 않은 평문 트래픽을 허용하도록 설정한다. 이는 네트워크 통신 시 데이터가 암호화되지 않아 중간에서 위험에 노출될 수 있다.

반면에 `android:networkSecurityConfig="@xml/network_security_config"` 속성은 앱의 네트워크 보안 구성을 세밀하게 제어할 수 있는 방법을 제공한다. 이를 통해 특정 도메인에 대해서만 평문 트래픽을 허용하거나, 신뢰할 수 있는 인증서를 지정하는 등 보다 세밀하고 안전한 네트워크 정책을 설정할 수 있다.

[관련 공식문서](https://developer.android.com/privacy-and-security/security-config?hl=ko)
