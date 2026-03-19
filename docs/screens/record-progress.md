# 독서 진행 상황 기록 (Record Reading Progress)

[:fontawesome-solid-arrow-up-right-from-square: Stitch에서 보기](https://stitch.withgoogle.com/projects/9973353658769610659){ target="_blank" .md-button }

---

## UI 구성

| 구성 요소 | 설명 |
|---|---|
| 책 표지 & 기본 정보 | 표지 이미지, 제목, 저자 |
| 시작 페이지 입력 | 숫자 키보드 |
| 끝 페이지 입력 | 숫자 키보드, 시작 페이지 이하 방지 |
| 독서 소요 시간 | 선택, 분 단위 |
| 날짜 표시 | 오늘 기본값, DatePicker로 변경 가능 |
| 저장 버튼 | 저장 후 이전 화면으로 복귀 |

---

## MVI 구조

```kotlin
data class RecordProgressUiState(
    val book: BookItemUiModel? = null,
    val startPage: String = "",
    val endPage: String = "",
    val durationMinutes: String = "",
    val selectedDate: LocalDate = today(),
    val progressPercent: Float = 0f,
    val endPageError: String? = null,
    val isLoading: Boolean = false,
)

sealed interface RecordProgressAction {
    data class StartPageChange(val value: String) : RecordProgressAction
    data class EndPageChange(val value: String) : RecordProgressAction
    data class DurationChange(val value: String) : RecordProgressAction
    data class DateChange(val date: LocalDate) : RecordProgressAction
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
    startPage = startPage.toInt(),
    endPage = endPage.toInt(),
    durationMinutes = durationMinutes.toIntOrNull(),
    sessionDate = selectedDate,
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
- [ ] 책 표지 + 기본 정보 헤더
- [ ] 시작 페이지 / 끝 페이지 입력 필드 (숫자 키보드)
- [ ] 끝 페이지 > 시작 페이지 유효성 검사
- [ ] 독서 소요 시간 입력 (선택, 분 단위)
- [ ] 날짜 표시 (오늘 날짜 기본값, DatePicker로 변경 가능)
- [ ] 저장 버튼
- [ ] `RecordProgressViewModel` / MVI State, Action, SideEffect 정의
- [ ] 진행률 자동 계산 `(endPage / totalPages * 100)`
- [ ] 완독 감지: `endPage >= totalPages` 시 `ReadingStatus.FINISHED` 자동 전환
- [ ] 완독 시 축하 다이얼로그 또는 애니메이션 표시
- [ ] `BookRepository.upsertBook()` + `ReadingSessionRepository.finishSession()` 연동
- [ ] Analytics: `reading_session_saved`, `book_completed` (완독 시) 이벤트 로깅
