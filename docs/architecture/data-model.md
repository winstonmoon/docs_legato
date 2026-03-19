# 데이터 모델

## 도메인 모델

### Book

```kotlin
data class Book(
    val id: String,           // UUID
    val title: String,
    val author: String,
    val totalPages: Int?,
    val status: ReadingStatus,
    val coverUri: String?,    // 이미지 URL 또는 로컬 경로
    val publisher: String?,
    val isbn: String?,
    val startedAt: Instant?,
    val finishedAt: Instant?,
    val currentPage: Int = 0,
)

enum class ReadingStatus {
    WISHLIST,   // 읽고 싶은
    READING,    // 읽는 중
    FINISHED,   // 완독
    PAUSED,     // 일시 중단
    DROPPED,    // 포기
}
```

### ReadingSession

```kotlin
data class ReadingSession(
    val id: String,
    val bookId: String,
    val startedAt: Instant,
    val endedAt: Instant?,
    val startPage: Int?,
    val endPage: Int?,
    val durationMinutes: Int?,
    val memo: String?,
    val sessionDate: LocalDate,
)
```

### BookNote

```kotlin
data class BookNote(
    val id: String,
    val bookId: String,
    val page: Int?,
    val content: String,
    val createdAt: Instant,
)
```

### UserPreference

```kotlin
data class UserPreference(
    val themeMode: ThemeMode,
    val languageTag: String?,  // "ko", "en", "ja", null = 시스템
)

enum class ThemeMode { SYSTEM, LIGHT, DARK }
```

---

## Room Entity

### BookEntity

```kotlin
@Entity(tableName = "books")
data class BookEntity(
    @PrimaryKey val id: String,
    val title: String,
    val author: String,
    val totalPages: Int?,
    val status: String,        // ReadingStatus.name
    val coverUri: String?,
    val publisher: String?,
    val isbn: String?,
    val startedAt: Long?,      // Instant.toEpochMilliseconds()
    val finishedAt: Long?,
    val currentPage: Int = 0,
    val createdAt: Long,
)
```

### ReadingSessionEntity

```kotlin
@Entity(
    tableName = "reading_sessions",
    foreignKeys = [ForeignKey(
        entity = BookEntity::class,
        parentColumns = ["id"],
        childColumns = ["bookId"],
        onDelete = ForeignKey.CASCADE,
    )]
)
data class ReadingSessionEntity(
    @PrimaryKey val id: String,
    val bookId: String,
    val startedAt: Long,
    val endedAt: Long?,
    val startPage: Int?,
    val endPage: Int?,
    val durationMinutes: Int?,
    val memo: String?,
    val sessionDate: String,   // LocalDate.toString() "yyyy-MM-dd"
)
```

---

## Supabase 테이블

| 테이블 | 컬럼 | 설명 |
|---|---|---|
| `users` | id, email, display_name, avatar_url | 사용자 프로필 |
| `books` | id, user_id, title, author, publisher, total_pages, cover_url, isbn, status, current_page, started_at, finished_at | 도서 정보 |
| `reading_sessions` | id, book_id, start_page, end_page, duration_min, session_date, memo, created_at | 독서 세션 |

!!! note "로컬 우선 전략"
    1단계: Room KMP 로컬 DB만 사용 (오프라인 우선)
    2단계: Supabase와 동기화 옵션 추가 예정

---

## Repository 인터페이스

```kotlin
interface BookRepository {
    fun observeBooks(status: ReadingStatus? = null): Flow<List<Book>>
    fun observeBook(bookId: String): Flow<Book?>
    suspend fun upsertBook(book: Book)
    suspend fun deleteBook(bookId: String)
}

interface ReadingSessionRepository {
    suspend fun startSession(bookId: String, startPage: Int?): String
    suspend fun finishSession(
        sessionId: String,
        endPage: Int,
        durationMinutes: Int? = null,
        memo: String? = null,
        sessionDate: LocalDate,
    )
    fun observeRecentSessions(limit: Int = 10): Flow<List<ReadingSession>>
    fun observeSessionsForBook(bookId: String): Flow<List<ReadingSession>>
}

interface StatsRepository {
    fun observeWeeklyStats(): Flow<WeeklyStats>
    fun observeMonthlyStats(month: YearMonth): Flow<MonthlyStats>
}

interface PreferenceRepository {
    fun observeThemeMode(): Flow<ThemeMode>
    suspend fun setThemeMode(mode: ThemeMode)
    fun observeLanguage(): Flow<String?>
    suspend fun setLanguage(tag: String?)
}
```
