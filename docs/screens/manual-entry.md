# 수동 도서 입력 (Manual Book Entry)

[:fontawesome-solid-arrow-up-right-from-square: Stitch에서 보기](https://stitch.withgoogle.com/projects/9973353658769610659){ target="_blank" .md-button }

---

## UI 구성

| 필드 | 필수 여부 | 키보드 타입 |
|---|---|---|
| 책 제목 | ✅ 필수 | 텍스트 |
| 저자 | ✅ 필수 | 텍스트 |
| 전체 페이지 수 | ✅ 필수 | 숫자 |
| 출판사 | 선택 | 텍스트 |
| ISBN | 선택 | 숫자 |
| 표지 이미지 URL | 선택 | URL |

- 필수 항목 미입력 시 인라인 에러 메시지 표시
- "Add Book" 버튼: 모든 필수 항목 입력 시 활성화
- 뒤로가기 / 취소: 확인 다이얼로그 없이 즉시 뒤로

---

## MVI 구조

```kotlin
data class ManualEntryUiState(
    val title: String = "",
    val author: String = "",
    val totalPages: String = "",
    val publisher: String = "",
    val isbn: String = "",
    val coverUrl: String = "",
    val titleError: String? = null,
    val authorError: String? = null,
    val totalPagesError: String? = null,
    val isLoading: Boolean = false,
) {
    val isSubmitEnabled: Boolean
        get() = title.isNotBlank() && author.isNotBlank() && totalPages.isNotBlank()
}

sealed interface ManualEntryAction {
    data class TitleChange(val value: String) : ManualEntryAction
    data class AuthorChange(val value: String) : ManualEntryAction
    data class TotalPagesChange(val value: String) : ManualEntryAction
    data class PublisherChange(val value: String) : ManualEntryAction
    data class IsbnChange(val value: String) : ManualEntryAction
    data class CoverUrlChange(val value: String) : ManualEntryAction
    data object SubmitClick : ManualEntryAction
}

sealed interface ManualEntrySideEffect {
    data object NavigateBack : ManualEntrySideEffect
    data class ShowError(val message: String) : ManualEntrySideEffect
}
```

---

## Repository 연동

```kotlin
bookRepository.upsertBook(
    Book(
        id = uuid(),
        title = state.title,
        author = state.author,
        totalPages = state.totalPages.toInt(),
        status = ReadingStatus.WISHLIST,
        coverUri = state.coverUrl.ifBlank { null },
    )
)
```

---

## Analytics 이벤트

| 이벤트 | 파라미터 |
|---|---|
| `book_added` | `source = "manual"` |

---

## TODO

- [ ] `ManualBookEntryScreen` Composable 구현 (Dark / Light 공통)
- [ ] 입력 폼 필드 구현
    - [ ] 책 제목 (필수, 유효성 검사)
    - [ ] 저자 (필수, 유효성 검사)
    - [ ] 전체 페이지 수 (필수, 숫자 키보드)
    - [ ] 출판사 (선택)
    - [ ] ISBN (선택)
    - [ ] 표지 이미지 URL 입력 (선택)
- [ ] 필수 항목 미입력 시 인라인 에러 메시지 표시
- [ ] "Add Book" 버튼 (모든 필수 항목 입력 시 활성화)
- [ ] 뒤로가기 / 취소 처리
- [ ] `ManualBookEntryViewModel` / MVI State, Action, SideEffect 정의
- [ ] `BookRepository.upsertBook()` 연동 (source = manual)
- [ ] Analytics: `book_added(source=manual)` 이벤트 로깅
