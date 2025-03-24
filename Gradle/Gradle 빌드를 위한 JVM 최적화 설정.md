CI 빌드 및 Gradle 성능을 향상시키기 위해 [gradle.properties](http://gradle.properties) 파일에서 JVM 옵션을 최적화하는 방법을 배웠다.

```groovy
org.gradle.jvmargs=\
  -Xmx4096m \
  -Xms512m \
  -XX:MaxHeapFreeRatio=40 -XX:MinHeapFreeRatio=20 \
  -XX:+UseG1GC \
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=/tmp/heapdump.hprof \
  -Xlog:gc* \
  -XX:+ExitOnOutOfMemoryError \
  -XX:+DisableExplicitGC \
  -XX:+AlwaysPreTouch \
  -XX:+TieredCompilation \
  -Dfile.encoding=UTF-8
```

**각 옵션의 역할**

1. **`-Xmx4096m`**
    - 최대 힙 크기를 4096MB(4GB)로 설정하여, 빌드 과정에서 메모리 부족으로 인한 오류를 예방한다.
2. **`-Xms512m`**
    - JVM 시작 시 초기 힙 크기를 512MB로 할당하여, 초기 메모리 할당 비용을 줄인다.
3. **`-XX:MaxHeapFreeRatio=40 -XX:MinHeapFreeRatio=20`** 
    - 힙의 여유 공간 비율을 조정한다다. 사용되지 않는 메모리가 일정 비율 이상이면 힙을 축소하고, 부족하면 확장하여 메모리 활용을 최적화한다.
4. **`-XX:+UseG1GC`** 
    - 최신 GC 알고리즘인 G1 가비지 컬렉터를 사용한다. G1GC는 대규모 힙에서도 짧은 정지 시간을 보장해준다.
5. **`-XX:+HeapDumpOnOutOfMemoryError`**
    - Out Of Memory(OOM) 오류 발생 시 힙 덤프를 생성하여, 문제 원인을 진단할 수 있도록 돕는다.
6. **`-XX:HeapDumpPath=/tmp/heapdump.hprof`**
    - 생성된 힙 덤프 파일을 /tmp/heapdump.hprof 경로에 저장한다.
7. **`-Xlog:gc***`
    - GC(가비지 컬렉션) 로그를 최신 방식(-Xlog)을 사용해 기록한다. 이전의 -XX:+PrintGCDetails는 더 이상 권장되지 않는다.
8. **`-XX:+ExitOnOutOfMemoryError`**
    - OOM 오류 발생 시 JVM을 즉시 종료시켜, 불안정한 상태로 계속 실행되는 것을 방지한다.
9. **`-XX:+DisableExplicitGC`**
    - 코드에서 호출하는 System.gc()를 무시하여, 불필요한 GC 호출로 인한 성능 저하를 방지한다.
10. **`-XX:+AlwaysPreTouch`**
    - JVM 시작 시 메모리 페이지를 미리 할당(pre-touch)하여, 런타임 시 페이지 폴트(page fault) 발생을 줄인다.
11. **`-XX:+TieredCompilation`**
    - 여러 단계의 컴파일(Tiered Compilation)을 활성화하여, 초기 빠른 컴파일과 이후 최적화된 코드를 모두 사용할 수 있게 한다.
12. **`-Dfile.encoding=UTF-8`**
    - 파일 인코딩을 UTF-8로 설정하여, 국제화 및 다양한 문자 처리를 안정적으로 지원한다.

**추가) GC 옵션 충돌 피하기**

- `G1GC`와 `UseParallelGC` 옵션은 동시에 사용할 수 없으므로, 최신 JVM 환경에서는 G1GC를 선택하고 UseParallelGC 옵션은 제거했다.
