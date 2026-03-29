# 독서 기록 (Log Reading)

[:fontawesome-solid-arrow-up-right-from-square: Stitch에서 보기](https://stitch.withgoogle.com/projects/9973353658769610659){ target="_blank" .md-button }

---

## UI 구성

| 구성 요소 | 설명 |
|---|---|
| 책 정보 헤더 | 표지 이미지, 제목, 저자 |
| 현재 페이지 입력 | 숫자 키보드, 전체 페이지 수 초과 방지 |
| 전체 페이지 수 표시 | read-only |
| 진행률 바 | 페이지 입력 시 실시간 업데이트 |
| 날짜 선택 (DatePicker) | 기본값: 오늘, KMP 공용 또는 expect/actual |
| 메모 / 노트 입력 | 선택, 멀티라인 TextArea |
| "Save" 버튼 | 저장 후 이전 화면으로 복귀 |

---

## MVI 구조

```kotlin
data class LogReadingUiState(
    val book: BookItemUiModel? = null,
    val currentPage: String = "",
    val selectedDate: LocalDate = today(),
    val memo: String = "",
    val progressPercent: Float = 0f,
    val currentPageError: String? = null,
    val isLoading: Boolean = false,
)

sealed interface LogReadingAction {
    data class CurrentPageChange(val value: String) : LogReadingAction
    data class DateChange(val date: LocalDate) : LogReadingAction
    data class MemoChange(val value: String) : LogReadingAction
    data object SaveClick : LogReadingAction
}

sealed interface LogReadingSideEffect {
    data object NavigateBack : LogReadingSideEffect
    data class ShowError(val message: String) : LogReadingSideEffect
}
```

---

## 진행률 계산

```kotlin
val progressPercent = (currentPage.toIntOrNull() ?: 0)
    .coerceAtMost(book.totalPages)
    .toFloat() / book.totalPages
```

---

## Repository 연동

```kotlin
readingSessionRepository.finishSession(
    sessionId = activeSession.id,
    endPage = currentPage.toInt(),
    memo = memo,
    sessionDate = selectedDate,
)
bookRepository.upsertBook(
    book.copy(currentPage = currentPage.toInt())
)
```

---

## Analytics 이벤트

| 이벤트 | 파라미터 |
|---|---|
| `reading_session_saved` | `pages_read`, `book_id` |

---

## TODO

- [x] `LogReadingScreen` Composable 구현 (Light 우선, Dark 추가)
- [ ] 책 정보 헤더 구현 (표지 이미지, 제목, 저자)
- [ ] 현재 페이지 입력 필드 (숫자 키보드, 전체 페이지 수 초과 방지)
- [x] 진행률 `LinearProgressIndicator` (페이지 입력 시 실시간 업데이트)
- [ ] DatePicker 구현 (KMP 공용 또는 expect/actual)
- [ ] 메모 / 노트 입력 TextArea (선택)
- [ ] "Save" 버튼
- [x] `LogReadingViewModel` / MVI State, Action, SideEffect 정의
- [ ] `ReadingSessionRepository.finishSession()` 연동
- [ ] 저장 완료 후 Home / Library 데이터 자동 갱신
- [ ] Analytics: `reading_session_saved` 이벤트 로깅
