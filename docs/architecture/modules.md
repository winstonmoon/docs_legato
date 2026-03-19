# 모듈 구조

## 디렉토리 트리

```
composeApp/src/
  commonMain/kotlin/com/winstonmoon/legato/
    core/
      designsystem/    ← 테마, 타이포그래피, 공용 컴포넌트
      model/           ← 도메인 모델 (Book, ReadingSession 등)
      data/            ← Repository 구현체, Mapper, DataSource
      database/        ← Room DB, DAO, Entity, Migration
      domain/          ← UseCase
      navigation/      ← AppRoute, AppNavigator 인터페이스
      localization/    ← Locale 상태, 언어 변경 정책
    feature/
      login/           ← 로그인
      home/            ← 홈 대시보드
      library/         ← 도서관/히스토리
      search/          ← 검색 & 추가
      profile/         ← 프로필 & 설정
      reading/         ← 독서 기록 / 진행 상황

  androidMain/kotlin/
    navigation/        ← Android NavHost, BottomNavigation, Rail, Drawer
    platform/          ← Android 전용 DI, expect/actual 구현

  iosMain/kotlin/
    navigation/        ← iOS Stack Navigator 구현
    platform/          ← iOS 전용 DI, expect/actual 구현
```

---

## 패키지 역할 상세

### `core/model`
앱 전역에서 사용하는 순수 도메인 모델. 플랫폼 의존성 없음.

```kotlin
data class Book(...)
data class ReadingSession(...)
data class BookNote(...)
data class UserPreference(...)
enum class ReadingStatus { WISHLIST, READING, FINISHED, PAUSED, DROPPED }
enum class ThemeMode { SYSTEM, LIGHT, DARK }
```

### `core/data`
Repository 구현체, DataSource, Mapper.

```
BookRepository (interface in domain, impl here)
ReadingSessionRepository
StatsRepository
PreferenceRepository
```

### `core/database`
Room KMP 설정.

```kotlin
@Database(entities = [BookEntity::class, ReadingSessionEntity::class, ...])
abstract class LegatoDatabase : RoomDatabase()
```

### `core/domain`
UseCase. Repository를 조합하는 비즈니스 로직.

```
GetCurrentlyReadingBooksUseCase
SaveReadingSessionUseCase
GetWeeklyStatsUseCase
```

### `core/navigation`
공용 라우트 정의 및 Navigator 인터페이스.

```kotlin
sealed interface AppRoute {
    data object Home : AppRoute
    data object Library : AppRoute
    data object Search : AppRoute
    data object Profile : AppRoute
    data class BookDetail(val bookId: String) : AppRoute
    data class RecordProgress(val bookId: String) : AppRoute
}

interface AppNavigator {
    val currentRoute: StateFlow<AppRoute>
    fun navigate(route: AppRoute)
    fun back()
}
```

### `core/designsystem`
테마, 타이포그래피, 아이콘, 재사용 컴포넌트.

### `feature/*`
화면별 Composable, ViewModel, UiState, Action, SideEffect.

```
feature/home/
  HomeScreen.kt
  HomeViewModel.kt
  HomeUiState.kt   ← State / Action / SideEffect 정의
```

---

## 다국어 리소스 구조

```
composeApp/src/commonMain/composeResources/
  values/strings.xml         ← 기본 (영어)
  values-ko/strings.xml      ← 한국어
  values-ja/strings.xml      ← 일본어
  font/Inter-Regular.ttf
  font/Inter-Medium.ttf
  font/Inter-SemiBold.ttf
  drawable/ic_smiling_book.xml
```
