기본적으로 gradle은 증분 빌드(incremental build)를 지향한다. 증분 빌드는 재빌드를 할 때 이전 빌드와 비교하여 다시 빌드할 필요성이 있는지 없는지 판단하여 필요한 부분만 다시 빌드하는 것을 말한다. 이 때 필요한 부분만 다시 빌드할 수 있도록 local에 cahching 해두는 것을 build cache라고 한다.  

<br/>

```kotlin
org.gradle.caching=true
```

<br/>

위와 같은 코드를 gradle.properties에 추가하여 빌드 캐싱을 기본적으로 설정되게 한다. 

<br/>

```kotlin
kapt.incremental.apt=true
kotlin.incremental=true
```

<br/>

추가로 위 설정을 추가하여, kapt와 코틀린 컴파일러도 증분 빌드가 가능하도록 허용할 수 있다. 

증분 빌드를 제외하고 추가로 빌드 속도를 최적화하는 코드를 작성해보겠다. 

<br/>

### **1. Gradle 데몬 활성화**

```
org.gradle.daemon=true
```

- Gradle 데몬을 활성화하면 빌드 초기화 시간을 줄인다.
- 반복적으로 빌드하는 프로젝트에서 특히 유용합니다.

<br/>

### **2. 병렬 빌드 활성화**

```
org.gradle.parallel=true
```

- 병렬 빌드를 활성화하여 CPU 코어를 최대한 활용한다.
- 멀티모듈 프로젝트에서 빌드 시간을 단축한다.

<br/>

### **3. 필요 시 모듈만 구성**

```
org.gradle.configureondemand=true
```

- 필요한 모듈만 로드하고 구성하여 빌드 초기화를 최적화한다.

모든 경우에 항상 빌드 속도가 빨라지는 것은 아니다. 모듈의 개수와 같은 프로젝트 특성을 잘 파악하고 적용해야 한다.
