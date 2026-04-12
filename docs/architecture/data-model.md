# 데이터 모델

## 도메인 모델

### Book (Google Books API 검색 결과)

Google Books API 검색 결과를 나타내는 도메인 모델입니다. `authors`는 복수 저자를 지원하는 `List<String>` 형태입니다.

```kotlin
data class Book(
    val id: String,
    val title: String,
    val authors: List<String>,   // Google Books API: 복수 저자 목록
    val averageRating: Double?,
    val ratingsCount: Int?,
    val thumbnailUrl: String?,
) {
    val primaryAuthor: String get() = authors.firstOrNull() ?: "Unknown Author"
    val ratingDisplay: String get() = averageRating?.toString() ?: "-"
    val ratingsCountDisplay: String get() = ratingsCount?.let { formatCount(it) } ?: ""
}
```

### BookItemUiModel

`BookEntity`를 UI 레이어에서 사용하기 위한 변환 모델입니다. `genre`, `notes`는 목록 뷰에 불필요하여 의도적으로 제외됩니다.

```kotlin
data class BookItemUiModel(
    val id: String,
    val title: String,
    val author: String,
    val thumbnailUrl: String?,
    val status: ReadingStatus,
    val currentPage: Int,
    val totalPages: Int,
    val rating: Float?,
    val finishedDate: String?,
) {
    val progressPercent: Float
        get() = if (totalPages > 0) currentPage.toFloat() / totalPages else 0f
    val progressPercentInt: Int
        get() = (progressPercent * 100).toInt()
}
```

### ReadingSession

```kotlin
data class ReadingSession(
    val id: String,
    val bookId: String,
    val pagesRead: Int,          // 이번 세션에서 읽은 페이지 수
    val sessionDate: String,     // "yyyy-MM-dd"
    val memo: String?,
    val mood: String?,           // ReadingMood.name
)
```

### ReadingMood

```kotlin
enum class ReadingMood {
    BORING, OKAY, GOOD, GREAT
}
```

### ReadingStats

```kotlin
data class ReadingStats(
    val booksRead: Int = 0,
    val pagesRead: Int = 0,
    val dayStreak: Int = 0,
)
```

### ReadingStatus

```kotlin
enum class ReadingStatus {
    WISHLIST,   // 읽고 싶은
    READING,    // 읽는 중
    FINISHED,   // 완독
    PAUSED,     // 일시 중단
    DROPPED,    // 포기
}
```

### UserPreference

```kotlin
data class UserPreference(
    val themeMode: ThemeMode,
    val languageTag: String,       // "ko", "en", "ja", "" = 시스템 기본값 (KMP iOS 환경에서 null 대신 빈 문자열 사용)
    val authState: AuthState,      // 인증 상태 (UNAUTHENTICATED / GUEST / AUTHENTICATED)
    val currentUserId: String?,    // 게스트 포함 모든 인증 상태에서 userId 보유. 미인증 시 null
)

enum class ThemeMode { SYSTEM, LIGHT, DARK }

/**
 * 앱 내 인증 상태를 3단계로 구분한다.
 *
 * - UNAUTHENTICATED: 앱 최초 설치 또는 로그아웃 후 상태. 로그인 화면 표시.
 * - GUEST: "Continue as Guest" → [나중에] 선택 시. Supabase `signInAnonymously()`로 익명 userId 발급.
 *          로컬 DB 전용 (서버 동기화 없음). 앱 재시작 시 홈으로 자동 이동.
 * - AUTHENTICATED: 이메일 / Google / Apple 로그인 또는 게스트에서 `linkIdentity()` 전환 완료.
 *                   로컬 DB + Supabase 동기화 (Local-first).
 *
 * 게스트 → 회원 전환: `supabaseClient.auth.linkIdentity()` 사용.
 * userId가 유지되므로 `BookEntity` / `ReadingSessionEntity`의 userId 컬럼 마이그레이션 불필요.
 */
enum class AuthState { UNAUTHENTICATED, GUEST, AUTHENTICATED }
```

!!! note "BookNote (Phase 2 예정, 현재 미구현)"
    하이라이트·메모 기능(`BookNote` 엔티티)은 현재 미구현 상태입니다. Phase 2에서 추가될 예정입니다.

---

## Room Entity

### BookEntity

로컬 DB에 저장되는 엔티티입니다. `author`는 단일 문자열로 저장됩니다 (Google Books API의 `authors` 목록과 달리 단일 `String`).

```kotlin
@Entity(tableName = "book")
data class BookEntity(
    @PrimaryKey val id: String,
    @ColumnInfo(name = "user_id") val userId: String,        // 데이터 귀속 userId (게스트 포함)
    val title: String,
    val author: String,                                      // 로컬 DB: 단일 문자열
    @ColumnInfo(name = "thumbnail_url") val thumbnailUrl: String? = null,
    @ColumnInfo(name = "total_pages") val totalPages: Int = 0,
    @ColumnInfo(name = "current_page") val currentPage: Int = 0,
    val status: String = "WISHLIST",                        // ReadingStatus.name
    val rating: Float? = null,
    @ColumnInfo(name = "finished_date") val finishedDate: String? = null,
    val genre: String? = null,
    val notes: String? = null,
    @ColumnInfo(name = "created_at") val createdAt: Long = 0,
)
```

!!! note "userId 컬럼"
    `userId`는 Supabase `signInAnonymously()`로 발급된 익명 userId 포함 모든 인증 상태에서 존재한다.
    게스트 → 회원 전환 시 `linkIdentity()`를 사용하므로 userId가 유지되어 마이그레이션이 불필요하다.
    Room migration: `ALTER TABLE book ADD COLUMN user_id TEXT NOT NULL DEFAULT ''`

### ReadingSessionEntity

```kotlin
@Entity(tableName = "reading_session")
data class ReadingSessionEntity(
    @PrimaryKey val id: String,
    @ColumnInfo(name = "user_id") val userId: String,        // 데이터 귀속 userId (게스트 포함)
    @ColumnInfo(name = "book_id") val bookId: String,
    @ColumnInfo(name = "pages_read") val pagesRead: Int = 0,  // 이번 세션에서 읽은 페이지 수
    @ColumnInfo(name = "session_date") val sessionDate: String, // "yyyy-MM-dd"
    val memo: String? = null,
    val mood: String? = null,                                   // ReadingMood.name
    @ColumnInfo(name = "created_at") val createdAt: Long = 0,
)
```

---

## Supabase 테이블

| 테이블 | 컬럼 | 설명 |
|---|---|---|
| `users` | id, email, display_name, avatar_url | 사용자 프로필 |
| `books` | id, user_id, title, author, publisher, total_pages, cover_url, isbn, status, current_page, started_at, finished_at | 도서 정보 |
| `reading_sessions` | id, book_id, pages_read, session_date, memo, mood, created_at | 독서 세션 |

!!! note "Local-first 동기화 전략"
    - **게스트 (`AuthState.GUEST`)**: 로컬 DB 전용. 서버 동기화 없음. 기기 변경 / 앱 삭제 시 데이터 소실.
    - **회원 (`AuthState.AUTHENTICATED`)**: 로컬 DB에 즉시 저장 → 백그라운드에서 Supabase 업로드.
      오프라인 상태에서도 정상 동작하며, 온라인 복귀 시 미동기화 데이터 자동 업로드.
    - **게스트 → 회원 전환**: `linkIdentity()`로 userId 유지 → 기존 로컬 데이터 자동 귀속, 별도 마이그레이션 불필요.

---

## Repository 인터페이스

### BookRepository

Google Books API 검색 전용입니다.

```kotlin
interface BookRepository {
    suspend fun searchBooks(query: String): List<Book>
}
```

### LocalBookRepository

Room CRUD 및 통계 조회를 담당합니다. 모든 조회는 `Flow` 기반으로 동작합니다.

```kotlin
interface LocalBookRepository {
    fun observeAllBooks(): Flow<List<BookEntity>>
    fun observeBooksByStatus(status: ReadingStatus): Flow<List<BookEntity>>
    fun observeBook(bookId: String): Flow<BookEntity?>
    fun observeReadingStats(): Flow<ReadingStats>   // StatsRepository 역할 통합
    suspend fun insertBook(book: BookEntity)
    suspend fun updateProgress(bookId: String, currentPage: Int)
    suspend fun updateRating(bookId: String, rating: Float?)
    suspend fun updateStatus(bookId: String, status: ReadingStatus)
    suspend fun deleteBook(bookId: String)
}
```

!!! note "StatsRepository 없음"
    별도의 `StatsRepository`는 존재하지 않습니다. 통계 조회는 `LocalBookRepository.observeReadingStats()`로 통합되어 있습니다.

### LocalReadingSessionRepository

독서 세션 저장 및 조회를 담당합니다.

```kotlin
interface LocalReadingSessionRepository {
    suspend fun insertSession(session: ReadingSessionEntity)
    fun observeSessionsByBookId(bookId: String): Flow<List<ReadingSessionEntity>>
}
```

### AppPreferences

테마·언어·인증 상태를 관리합니다. `StateFlow` 기반으로 동작합니다.

```kotlin
interface AppPreferences {
    val themeMode: StateFlow<ThemeMode>
    val languageTag: StateFlow<String>          // "" = 시스템 기본값
    val authState: StateFlow<AuthState>         // UNAUTHENTICATED / GUEST / AUTHENTICATED
    val currentUserId: StateFlow<String?>       // 게스트 포함 모든 인증 상태에서 userId 보유

    fun setThemeMode(mode: ThemeMode)
    fun setLanguageTag(tag: String)
    fun setAuthState(state: AuthState)
    fun setCurrentUserId(userId: String?)
}
```

!!! warning "`isLoggedIn: Boolean` 폐기"
    기존 `isLoggedIn: Boolean`은 게스트/회원을 구분하지 못하는 구조적 한계로 `AuthState` enum으로 대체됩니다.
    `LoginViewModel`의 `preferences.isLoggedIn.filter { it }` 패턴은 `preferences.authState.filter { it != AuthState.UNAUTHENTICATED }`로 수정이 필요합니다.
