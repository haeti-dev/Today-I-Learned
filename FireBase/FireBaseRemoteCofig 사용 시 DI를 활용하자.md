```kotlin
    companion object {
        private val remoteConfig by lazy {
            Firebase.remoteConfig.apply {
                setConfigSettingsAsync(
                    remoteConfigSettings {
                        minimumFetchIntervalInSeconds = 0
                    }
                )
            }
        }

        fun getRemoteConfigInstance(): FirebaseRemoteConfig = remoteConfig
    }

```

기존에는 위와 같이 ViewModel 에서 팩토리 메서드 패턴을 통해 싱글톤을 구현하였다. 

하지만 Remote Config 사용을 더 다양한 곳에서 할 것을 염두하면 DI를 통해 좀 더 캡슐화를 하고 전역적으로 사용할 수 있게 설정하면 좋을 것 같다. 

```kotlin
@Module
@InstallIn(SingletonComponent::class)
class FirebaseRemoteConfigDi {

    @Provides
    fun provideFirebaseRemoteConfig(): FirebaseRemoteConfig {
        val firebaseRemoteConfig: FirebaseRemoteConfig = FirebaseRemoteConfig.getInstance()
        firebaseRemoteConfig
            .setConfigSettingsAsync(
                FirebaseRemoteConfigSettings.Builder()
                    .setMinimumFetchIntervalInSeconds(2L)
                    .build(),
            )
        firebaseRemoteConfig.setDefaultsAsync(defaultValueMap) // Important!
        return firebaseRemoteConfig
    }

    @Provides
    fun provideFirebaseRemoteConfigProvider(
        firebaseRemoteConfig: FirebaseRemoteConfig
    ): FirebaseRemoteConfigProvider {
        return FirebaseRemoteConfigProvider(firebaseRemoteConfig)
    }
}
```

위와 같은 Hilt를 통한 DI를 통해 캡슐화를 시도할 수 있다. 또한 레퍼런스에 따르면 관리하는 값들을 Map 자료 구조로 관리하는 것이 용이해보이며, 기존에 하던 방식처럼 데이터 스트림을 구독하는 방식으로 구현하여 실시간 데이터를 반영하면 좋은 퍼포먼스를 낼 수 있을 것이라 생각한다. 

### 레퍼런스

https://proandroiddev.com/android-feature-flag-implementation-with-firebase-remote-config-kotlin-flow-jetpack-compose-79fc15194a42
