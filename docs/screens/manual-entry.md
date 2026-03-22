# 수동 도서 입력 (Manual Book Entry)

[:fontawesome-solid-arrow-up-right-from-square: Stitch에서 보기](https://stitch.withgoogle.com/projects/9973353658769610659){ target="_blank" .md-button }

---

## UI 구성

> Stitch 디자인: **Manual Book Entry (Light)** / **Manual Book Entry (Dark)**

### Stitch 스크린샷

=== "Light"
    ![Manual Entry Light](https://lh3.googleusercontent.com/aida/ADBb0ui08hnOETepcZBTdYJz6iqaWw5vfGSrgj_NHex99_k1NrbB8_VBPhTHke-fWSjKFxWHx0_Zg4kqOl8AsShFKHmt40nNOjmkH3VdvhCwLEuJ00XNBLCBk-Z7_2wOe04DEQDITalornmYRB8Fxj3Sr7Lra0bFdaLRv1hgRe0OfonMnmLJa3wTBh0drHk2F9F13jHkWU_BUSOZ3fiPGmePEhZrUrzO8_CHJzmjY2iJUxIlxCAiea2Wwkw_H1s)

=== "Dark"
    ![Manual Entry Dark](https://lh3.googleusercontent.com/aida/ADBb0uhTvaYub3CSgpZ-mvEX6yehMMD6pzNMqUmkfLdZuDM7wIF09jrP9og4-aPgwvJsyD5oneGkqHRCRY0qQbogXRxkTNpIwXq8uQr_0URGtNzfoVqQxHfeR4ygC-dEH8pxJ0zYHZaFBxi6jSeWRYio0mA_bMbfpUMFtNZgc0FznRpbl9gNezVk7VtfqT95QBqzla0PML_JwnGTbikIl3Il7J4bTINMko5noOOiLxsE6nQEjUshs8zo-FBo0g0)

### 입력 필드

| 필드 | 필수 여부 | 키보드 타입 |
|---|---|---|
| 책 표지 이미지 | 선택 | 갤러리 / 카메라 (파일 피커) |
| 책 제목 | ✅ 필수 | 텍스트 |
| 저자 | ✅ 필수 | 텍스트 |
| 전체 페이지 수 | ✅ 필수 | 숫자 |
| 장르 (Category) | 선택 | 드롭다운 선택 |
| 장르 태그 (Quick Select) | 선택 | 탭 토글 칩 |
| Notes | 선택 | 멀티라인 텍스트 |

### 장르 드롭다운 옵션

Fiction / Non-Fiction / Mystery / Fantasy / Sci-Fi / Biography

### 장르 태그 퀵셀렉트 칩

`Classic` · `Drama` · `Romance` · `Historical` (미리 정의된 태그, 복수 선택 가능)

!!! note "동작 방식"
    - 장르 드롭다운과 태그 칩은 별개로 동작
    - 드롭다운: 도서의 메인 카테고리
    - 태그 칩: 세부 분류 (멀티 선택)
    - 필수 항목 미입력 시 인라인 에러 메시지 표시
    - "Add to Library" 버튼: 제목 · 저자 · 페이지 수 입력 시 활성화
    - Cancel / 뒤로가기: 확인 다이얼로그 없이 즉시 뒤로

---

## MVI 구조

```kotlin
data class ManualEntryUiState(
    val title: String = "",
    val author: String = "",
    val totalPages: String = "",
    val genre: BookGenre? = null,
    val selectedTags: Set<String> = emptySet(),
    val notes: String = "",
    val coverImageUri: String? = null,  // 갤러리/카메라에서 선택한 로컬 URI
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
    data class GenreChange(val genre: BookGenre?) : ManualEntryAction
    data class TagToggle(val tag: String) : ManualEntryAction
    data class NotesChange(val value: String) : ManualEntryAction
    data class CoverImageSelected(val uri: String) : ManualEntryAction
    data object SubmitClick : ManualEntryAction
    data object CancelClick : ManualEntryAction
}

sealed interface ManualEntrySideEffect {
    data object NavigateBack : ManualEntrySideEffect
    data object OpenImagePicker : ManualEntrySideEffect  // 갤러리/카메라 선택
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
        genre = state.genre,
        tags = state.selectedTags.toList(),
        notes = state.notes.ifBlank { null },
        status = ReadingStatus.WISHLIST,
        coverUri = state.coverImageUri,
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
- [ ] 표지 이미지 업로드 영역 구현
    - [ ] 점선 테두리 드롭존 + `add_a_photo` 아이콘
    - [ ] "Select Image" 버튼 탭 → 갤러리 / 카메라 선택 bottomsheet
    - [ ] 선택 후 표지 미리보기 표시
- [ ] 입력 폼 필드 구현
    - [ ] 책 제목 (필수, 유효성 검사)
    - [ ] 저자 (필수, 유효성 검사)
    - [ ] 전체 페이지 수 (필수, 숫자 키보드)
    - [ ] 장르 드롭다운 (Fiction / Non-Fiction / Mystery / Fantasy / Sci-Fi / Biography)
    - [ ] 장르 태그 퀵셀렉트 칩 (Classic · Drama · Romance · Historical, 멀티 선택)
    - [ ] Notes 텍스트 에어리어 (선택, 3줄)
- [ ] 필수 항목 미입력 시 인라인 에러 메시지 표시
- [ ] "Add to Library" 버튼 (모든 필수 항목 입력 시 활성화)
- [ ] Cancel / 뒤로가기 처리
- [ ] `ManualBookEntryViewModel` / MVI State, Action, SideEffect 정의
- [ ] `BookRepository.upsertBook()` 연동 (genre, tags, notes, coverUri 포함)
- [ ] Analytics: `book_added(source=manual)` 이벤트 로깅
