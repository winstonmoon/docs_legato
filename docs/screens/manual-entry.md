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
| 책 표지 이미지 | 선택 | 점선 테두리 드롭존 + `add_a_photo` 아이콘; 탭 시 갤러리 / 카메라 선택 BottomSheet |
| 책 제목 | ✅ 필수 | 텍스트 |
| 저자 | ✅ 필수 | 텍스트 |
| 전체 페이지 수 | ✅ 필수 | 숫자 |
| 장르 (Category) | 선택 | 드롭다운 선택 |
| 장르 태그 (Quick Select) | 선택 | 탭 토글 칩 |
| Notes | 선택 | 멀티라인 텍스트 |

### 장르 드롭다운 옵션 (12개)

Fiction / Non-Fiction / Mystery / Fantasy / Science Fiction / Biography / Self-Help / Business / History / Science / Romance / Poetry

### 장르 태그 퀵셀렉트 칩

`Classic` · `Drama` · `Romance` · `Historical` (미리 정의된 태그, 복수 선택 가능)

!!! note "동작 방식"
    - 장르 드롭다운과 태그 칩은 별개로 동작
    - 드롭다운: 도서의 메인 카테고리 (12개 옵션, Single-select)
    - 태그 칩: 세부 분류 (멀티 선택)
    - 필수 항목 미입력 시 인라인 에러 메시지 표시
    - "Add to Library" 버튼: `Scaffold bottomBar`에 고정 배치, 제목 · 저자 · 페이지 수 입력 시 활성화 + 로딩 인디케이터 표시
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

## 조사 결과: 필드 설계 분석

> 바코드 스캔이나 검색으로 도서를 찾지 못한 경우를 위한 수동 입력 화면이므로,
> Google Books API에서 자동 취득 가능한 정보도 이 화면에서는 **사용자가 직접 입력**해야 합니다.

### Book 데이터 모델 vs Google Books API 취득 가능 정보

| 데이터 모델 필드 | Google Books API 대응 | 비고 |
|---|---|---|
| `title` | `volumeInfo.title` | ✅ API 취득 가능 |
| `author` | `volumeInfo.authors[]` | ✅ API 취득 가능 |
| `totalPages` | `volumeInfo.pageCount` | ✅ API 취득 가능 |
| `coverUri` | `volumeInfo.imageLinks.thumbnail` | ✅ API 취득 가능 |
| `publisher` | `volumeInfo.publisher` | ✅ API 취득 가능 |
| `isbn` | `volumeInfo.industryIdentifiers[]` (ISBN_13/10) | ✅ API 취득 가능 |
| `status` | — | ❌ 사용자 입력 필요 |
| `currentPage` | — | 기본값 0 (앱 내부 관리) |
| `startedAt` | — | 상태 변경 시 자동 기록 |
| `finishedAt` | — | 상태 변경 시 자동 기록 |
| `id` | — | UUID 자동 생성 |

API에서 취득 가능하지만 **현재 데이터 모델에 없는** 유용한 필드:

| Google Books API 필드 | 설명 |
|---|---|
| `volumeInfo.categories[]` | 장르/카테고리 |
| `volumeInfo.publishedDate` | 출판일 |
| `volumeInfo.description` | 책 소개/시놉시스 |
| `volumeInfo.language` | 언어 (ISO 639-1) |

---

### Manual Entry 입력 항목 분석

#### 현재 구현된 항목

| 항목 | 필수 여부 | 데이터 모델 대응 |
|---|---|---|
| 책 표지 이미지 | 선택 | `coverUri` |
| 책 제목 | ✅ 필수 | `title` |
| 저자 | ✅ 필수 | `author` |
| 전체 페이지 수 | ✅ 필수 | `totalPages` |
| 장르 (Category) | 선택 | 데이터 모델에 `categories` 필드 없음 → 추가 검토 필요 |
| 장르 태그 | 선택 | 데이터 모델에 `tags` 필드 없음 → 추가 검토 필요 |
| Notes | 선택 | 데이터 모델에 `notes` 필드 없음 → 추가 검토 필요 |

#### 누락된 항목 (데이터 모델에 존재하나 UI 없음)

| 항목 | 데이터 모델 필드 | 우선순위 |
|---|---|---|
| 출판사 | `publisher` | 중간 — 수동 입력 시 기록해두면 유용 |
| ISBN | `isbn` | 낮음 — 수동 입력이지만 나중에 API 연동 시 활용 가능 |
| 독서 상태 | `status` | 높음 — 추가 시 WISHLIST/READING 바로 선택 가능해 UX 개선 |

!!! warning "데이터 모델 보완 검토"
    - `categories` / `tags` 필드: UI에는 이미 구현되어 있으나 `Book` 도메인 모델에 해당 필드가 없음
    - `notes` 필드: UI에는 구현되어 있으나 `Book` 도메인 모델에 없음
    - 위 세 필드는 모델 추가 후 Repository 연동이 필요함

---

## TODO

- [x] `ManualBookEntryScreen` Composable 구현 (Dark / Light 공통)
- [ ] 표지 이미지 업로드 영역 구현 (미착수)
    - [ ] 점선 테두리 드롭존 + `add_a_photo` 아이콘
    - [ ] "Select Image" 버튼 탭 → 갤러리 / 카메라 선택 bottomsheet
    - [ ] 선택 후 표지 미리보기 표시
- [x] 입력 폼 필드 구현
    - [x] 책 제목 (필수, 유효성 검사)
    - [x] 저자 (필수, 유효성 검사)
    - [x] 전체 페이지 수 (필수, 숫자 키보드)
    - [x] 장르 드롭다운 (`DropdownMenu` + 퀵셀렉트 `AssistChip` 구현됨; `BookGenre` enum 전환 미완)
    - [ ] 장르 태그 멀티 선택 칩 (Classic · Drama · Romance · Historical) — 현재 단일 장르 선택만 구현
    - [x] Notes 텍스트 에어리어 (선택, 3줄)
- [x] 필수 항목 미입력 시 인라인 에러 메시지 표시
- [x] "Add to Library" 버튼 (`isSubmitEnabled` 조건부 활성화)
- [x] Cancel / 뒤로가기 처리
- [x] `ManualBookEntryViewModel` / MVI State, Action, SideEffect 정의
- [x] `BookRepository.insertBook()` 연동 (genre, notes 포함; tags·coverUri 미구현)
- [ ] Analytics: `book_added(source=manual)` 이벤트 로깅
