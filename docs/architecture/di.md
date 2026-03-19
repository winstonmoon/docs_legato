# DI 설계 (Koin)

## 모듈 구성

```kotlin
val appModules = listOf(
    platformModule,    // 플랫폼별 구현체
    databaseModule,    // Room DB, DAO
    repositoryModule,  // Repository 구현체
    featureModule,     // ViewModel, UseCase
)
```

---

## platformModule

플랫폼별로 분리 (androidMain / iosMain).

```kotlin
// androidMain
val platformModule = module {
    single<AppLogger> { AndroidLogger() }
    single<DatabaseBuilder> { AndroidDatabaseBuilder(androidContext()) }
    single<LocaleProvider> { AndroidLocaleProvider(androidContext()) }
}

// iosMain
val platformModule = module {
    single<AppLogger> { IosLogger() }
    single<DatabaseBuilder> { IosDatabaseBuilder() }
    single<LocaleProvider> { IosLocaleProvider() }
}
```

---

## databaseModule

```kotlin
val databaseModule = module {
    single<LegatoDatabase> {
        get<DatabaseBuilder>().build()
    }
    single<BookDao> { get<LegatoDatabase>().bookDao() }
    single<ReadingSessionDao> { get<LegatoDatabase>().readingSessionDao() }
    single<UserPreferenceDao> { get<LegatoDatabase>().userPreferenceDao() }
}
```

!!! warning "HttpClient 리소스 누수 방지"
    Ktor `HttpClient`를 Koin으로 관리할 때 반드시 `onClose`를 추가한다.
    ```kotlin
    single<HttpClient> {
        HttpClient(/* ... */)
    } onClose {
        it?.close()
    }
    ```

---

## repositoryModule

```kotlin
val repositoryModule = module {
    single<BookRepository> {
        BookRepositoryImpl(
            bookDao = get(),
            supabaseClient = get(),
        )
    }
    single<ReadingSessionRepository> {
        ReadingSessionRepositoryImpl(
            readingSessionDao = get(),
        )
    }
    single<StatsRepository> {
        StatsRepositoryImpl(readingSessionDao = get())
    }
    single<PreferenceRepository> {
        PreferenceRepositoryImpl(dataStore = get())
    }
}
```

---

## featureModule

```kotlin
val featureModule = module {
    // ViewModels
    viewModel { HomeViewModel(get(), get()) }
    viewModel { LibraryViewModel(get()) }
    viewModel { SearchViewModel(get(), get()) }
    viewModel { ManualBookEntryViewModel(get()) }
    viewModel { LogReadingViewModel(get(), get()) }
    viewModel { RecordProgressViewModel(get(), get()) }
    viewModel { ProfileViewModel(get(), get()) }
    viewModel { LoginViewModel(get()) }

    // UseCases
    factory { GetCurrentlyReadingBooksUseCase(get()) }
    factory { SaveReadingSessionUseCase(get(), get()) }
    factory { GetWeeklyStatsUseCase(get()) }
}
```

---

## Koin 초기화

=== "Android"

    ```kotlin
    class LegatoApplication : Application() {
        override fun onCreate() {
            super.onCreate()
            startKoin {
                androidLogger()
                androidContext(this@LegatoApplication)
                modules(appModules)
            }
        }
    }
    ```

=== "iOS"

    ```kotlin
    // iosMain
    fun initKoin() {
        startKoin {
            modules(appModules)
        }
    }
    ```

    ```swift
    // Swift
    @main
    struct iOSApp: App {
        init() {
            KoinHelperKt.initKoin()
        }
    }
    ```
