# 독서 진행 상황 기록 (Record Reading Progress)

[:fontawesome-solid-arrow-up-right-from-square: Stitch에서 보기](https://stitch.withgoogle.com/projects/9973353658769610659){ target="_blank" .md-button }

---

## UI 구성

> Stitch 디자인: **Record Reading Progress**

### Stitch 스크린샷

![Record Reading Progress](https://lh3.googleusercontent.com/aida/ADBb0uiSLJPi8sKkUFVAopKk3kazR-OSb40OxLI9t1Q0gE5HEBiUTSTxKgo1264f2IIjtv3RhIRUXMIzFQQuqH9SoSYY7I4gBR3iU0GISHVafR3CMj3fYSmN_E8MPKbEhzywLGmv8r619Owsf3eYhtq8URCI9z1uknCFn6yX2FRHZR_VumXIal1R_WkcY153afwgjPbxsu9Qiv-BjXF8bjnARRN5hYXokploMh7yFLV0n1T1fSaaJw7Hao4PzEA)

| 구성 요소 | 설명 |
|---|---|
| 책 표지 & 기본 정보 | 표지 이미지 (상단 진행률 바 오버레이 포함), 제목, 저자 |
| 진행률 표시 | 표지 하단에 오버레이된 진행률 바 + "X% Complete" 텍스트 |
| 오늘 읽은 페이지 스텝퍼 | `-` 버튼 · 숫자 입력 · `+` 버튼 (오늘 읽은 페이지 수) |
| 총 진행 페이지 표시 | "Total pages: 142 / 218" 형식 |
| Quick Notes | 텍스트 에어리어 + 이미지 첨부(`image`) / 태그 추가(`label`) 버튼 |
| How was it? | 독서 무드 선택: Boring · Okay · Good · Great |
| Save Log 버튼 | 하단 고정; 저장 후 이전 화면으로 복귀 |

### 독서 무드 (Mood)

| 무드 | 아이콘 | 설명 |
|---|---|---|
| Boring | `sentiment_dissatisfied` | 지루했음 |
| Okay | `sentiment_neutral` | 보통 |
| Good | `sentiment_satisfied` | 좋았음 |
| Great | `sentiment_very_satisfied` | 매우 좋았음 |

---

## MVI 구조

```kotlin
enum class ReadingMood { BORING, OKAY, GOOD, GREAT }

data class RecordProgressUiState(
    val book: BookItemUiModel? = null,
    val pagesReadToday: Int = 0,        // 스텝퍼로 입력하는 오늘 읽은 페이지 수
    val totalPagesRead: Int = 0,        // 누적 읽은 페이지 (book.currentPage 기반)
    val progressPercent: Float = 0f,
    val quickNote: String = "",
    val mood: ReadingMood? = null,
    val isLoading: Boolean = false,
)

sealed interface RecordProgressAction {
    data class PagesIncrement(val amount: Int = 1) : RecordProgressAction
    data class PagesDecrement(val amount: Int = 1) : RecordProgressAction
    data class PagesChange(val value: Int) : RecordProgressAction
    data class QuickNoteChange(val value: String) : RecordProgressAction
    data class MoodChange(val mood: ReadingMood) : RecordProgressAction
    data object SaveClick : RecordProgressAction
}

sealed interface RecordProgressSideEffect {
    data object NavigateBack : RecordProgressSideEffect
    data object ShowCompletionDialog : RecordProgressSideEffect // 완독 시
}
```

---

## 완독 감지 로직

```kotlin
val isCompleted = endPage.toIntOrNull() != null &&
    book.totalPages != null &&
    endPage.toInt() >= book.totalPages

if (isCompleted) {
    bookRepository.upsertBook(book.copy(status = ReadingStatus.FINISHED))
    sideEffect.send(RecordProgressSideEffect.ShowCompletionDialog)
}
```

---

## Repository 연동

```kotlin
readingSessionRepository.finishSession(
    bookId = book.id,
    pagesReadToday = state.pagesReadToday,
    note = state.quickNote.ifBlank { null },
    mood = state.mood,
    sessionDate = today(),
)
```

---

## Analytics 이벤트

| 이벤트 | 파라미터 | 조건 |
|---|---|---|
| `reading_session_saved` | `pages_read`, `book_id` | 항상 |
| `book_completed` | `book_id`, `total_pages` | 완독 시만 |

---

## TODO

- [ ] `RecordReadingProgressScreen` Composable 구현
- [ ] 책 표지 헤더 (진행률 바 + "X% Complete" 오버레이 포함)
- [ ] 책 제목 / 저자 텍스트
- [ ] 오늘 읽은 페이지 수 스텝퍼 (`-` 버튼 · 숫자 입력 · `+` 버튼)
- [ ] 총 진행 페이지 표시 ("Total pages: X / Y")
- [ ] Quick Notes 텍스트 에어리어 + 이미지/태그 첨부 버튼
- [ ] "How was it?" 무드 선택 버튼 4개 (Boring / Okay / Good / Great)
- [ ] Save Log 버튼 (하단 고정)
- [ ] `RecordProgressViewModel` / MVI State, Action, SideEffect 정의
- [ ] 진행률 자동 계산 `((currentPage + pagesReadToday) / totalPages * 100)`
- [ ] 완독 감지: 누적 페이지 >= totalPages 시 `ReadingStatus.FINISHED` 자동 전환
- [ ] 완독 시 축하 다이얼로그 또는 애니메이션 표시
- [ ] `BookRepository.upsertBook()` + `ReadingSessionRepository.finishSession()` 연동 (mood, note 포함)
- [ ] Analytics: `reading_session_saved`, `book_completed` (완독 시) 이벤트 로깅
