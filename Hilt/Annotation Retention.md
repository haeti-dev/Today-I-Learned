```kotlin
/**
 * Contains the list of possible annotation's retentions.
 *
 * Determines how an annotation is stored in binary output.
 */
public enum class AnnotationRetention {
    /** Annotation isn't stored in binary output */
    SOURCE,
    /** Annotation is stored in binary output, but invisible for reflection */
    BINARY,
    /** Annotation is stored in binary output and visible for reflection (default retention) */
    RUNTIME
}

```

`AnnotationRetention`은 위와 같이 3종류가 존재한다. 

<br/>

### 1. `AnnotationRentention.SOURCE`

- 생명주기 : 컴파일 전까지만 존재
- 바이트코드 : 포함되지 않음 (.class 파일에 없음)
- 리플렉션 불가능
- 사용 사례
    - IDE 힌트/경고
    - Lint 체커용 마커
- 장점 : 바이트코드 크기 증가 없음 ⇒ 성능 영향 0

### 2. `AnnotationRentention.BINARY`

- 생명주기 : 컴파일 후 .class 파일에 포함
- 바이트코드 : 포함됨
- 리플렉션 불가능
- 사용 사례
    - 컴파일 타임 프로세싱 (Annotation Processor)
    - 바이너리 호환성 유지
- 리플렉션 오버헤드 없음. 메타 데이터는 유지

### 3. `AnnotationRentention.RUNTIME`

- 생명주기 : 런타임까지 존재
- 바이트 코드 : 포함됨
- **리플렉션 가능**
- 사용 사례
    - DI
    - JUnit 테스트 어노테이션
    - Retrofit 어노테이션
    - Serialization
    - Qualifer
- 단점 : 약간의 메모리 오버헤드
