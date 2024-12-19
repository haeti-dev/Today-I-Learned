[관련 공식문서](https://developer.android.com/build/multidex?hl=ko)

## MultiDexApplication이란?

- `MultiDexApplication`은 **안드로이드의 MultiDex 지원**을 위한 기본 `Application` 클래스이다.
- 안드로이드 애플리케이션의 메서드 참조가 **64K(64,536) 제한**을 초과하는 경우, 여러 개의 DEX 파일을 사용해야 한다. 이를 지원하기 위해 MultiDex를 활성화해야 한다.
- `MultiDexApplication`은 MultiDex 초기화와 추가 DEX 파일 로드 작업을 자동으로 처리해주는 클래스이다.  

- Dex는 뭘까?
    
    DEX 파일이란 안드로이드 애플리케이션이 실행 가능한 **Dalvik Executable** 파일로, 앱 코드와 참조를 포함한다.
    

- 64k라는 숫자는 어디서 나온 걸까?
    
    Android 앱(APK) 파일에는 [Dalvik Executable(DEX)](https://source.android.com/devices/tech/dalvik/dex-format?hl=ko) 파일 형식의 실행 가능한 바이트 코드 파일이 포함되며, DEX 파일에는 앱을 실행하기 위해 사용되는 컴파일된 코드가 포함된다. Dalvik Executable 사양은 단일 DEX 파일 내에서 참조될 수 있는 메서드의 총 개수를 65,536개로 제한하며 여기에는 Android 프레임워크 메서드, 라이브러리 메서드, 자체 코드에 있는 메서드가 포함된다.
    
    컴퓨터 공학에서[*킬로 또는 K*](https://en.wikipedia.org/wiki/Kilo-)라는 용어는 1,024(또는 2^10)를 나타낸다. 65,536은 64 X 1,024와 동일하므로 이 제한을 '64K 참조 제한'이라고 부른다.

<br/>

## MultiDex가 필요한 이유

안드로이드 앱의 DEX 파일은 메서드 참조 제한(64K)을 가진다.

아래의 상황에서 이 제한을 초과할 가능성이 있다

1. **다수의 라이브러리를 포함하는 경우** : Firebase, Play Services, Jetpack 라이브러리 등을 다수 사용하는 경우
2. **복잡한 애플리케이션 구조** : 의존성 증가로 인해 메서드 수가 기하급수적으로 증가
3. **Legacy 코드 유지** : 이전 버전의 코드와 새로운 코드가 공존하는 경우

<br/>

## MultiDex 사용 시 주의사항

1. **앱 초기화 시간 증가**
    - 추가 DEX 파일 로드로 인해 앱 시작 시간이 늘어날 수 있다.
    - 초기 로드 시간을 줄이기 위해 불필요한 라이브러리를 제거하거나 프로가드를 사용해 코드 최적화를 권장한다고 한다.
2. **설치 문제**
    - Dalvik 런타임(API 21 이하)에서는 추가 DEX 파일의 로드가 제대로 되지 않는 경우가 있으므로 이를 테스트해야 한다.
3. **프로가드 최적화 권장**
    - 프로가드를 사용해 메서드 수를 줄이면 MultiDex 사용을 최소화할 수 있다.
