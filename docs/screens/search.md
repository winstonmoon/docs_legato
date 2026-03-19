# 검색 & 도서 추가 (Search & Add Book)

[:fontawesome-solid-arrow-up-right-from-square: Stitch에서 보기](https://stitch.withgoogle.com/projects/9973353658769610659){ target="_blank" .md-button }

---

## UI 구성

| 구성 요소 | 설명 |
|---|---|
| 검색 입력 필드 | "Search for a book…", 포커스 시 키보드 자동 올라오기 |
| 검색 결과 리스트 | 책 표지, 제목, 저자, 출판사/연도, "Add to Library" 버튼 |
| Empty State (No Results) | 안내 메시지 + "수동으로 입력하기" 링크 |
| Loading Indicator | API 요청 중 |
| Error State | 네트워크 오류 안내 |
| AdMob 배너 | 화면 하단 (비프리미엄 사용자) |

---

## 검색 상태

| 상태 | 설명 |
|---|---|
| `Initial` | 검색어 없음, 빈 화면 또는 최근 검색어 |
| `Loading` | Google Books API 요청 중 |
| `Results` | 검색 결과 목록 |
| `NoResults` | 결과 없음 → "수동으로 입력하기" 유도 |
| `Error` | 네트워크 오류 안내 메시지 |

---

## MVI 구조

```kotlin
data class SearchUiState(
    val query: String = "",
    val searchState: SearchState = SearchState.Initial,
)

sealed interface SearchState {
    data object Initial : SearchState
    data object Loading : SearchState
    data class Results(val books: List<SearchResultUiModel>) : SearchState
    data object NoResults : SearchState
    data class Error(val message: String) : SearchState
}

sealed interface SearchAction {
    data class QueryChange(val query: String) : SearchAction
    data class AddToLibraryClick(val book: SearchResultUiModel) : SearchAction
    data object ManualEntryClick : SearchAction
}

sealed interface SearchSideEffect {
    data object NavigateToManualEntry : SearchSideEffect
    data object ShowAddSuccessToast : SearchSideEffect
    data object ShowInterstitialAd : SearchSideEffect
}
```

---

## Google Books API 연동

```kotlin
// Debounce 처리 (300ms)
queryFlow
    .debounce(300)
    .filter { it.length >= 2 }
    .flatMapLatest { query ->
        bookApiService.searchBooks(query)
    }

// API 엔드포인트
GET https://www.googleapis.com/books/v1/volumes
  ?q={query}
  &key={API_KEY}
  &maxResults=20
```

!!! warning "API 키 보안"
    `local.properties`에 저장하고 `BuildConfig`를 통해 주입.
    `const val` 대신 `val`을 사용하여 APK 인라이닝 방지.

---

## AdMob — 도서 추가 완료 후 Interstitial

```kotlin
// 도서 추가 성공 시
addCount++
if (addCount % 3 == 0) { // 3회마다 1번
    showInterstitialAd()
}
```

---

## Analytics 이벤트

| 이벤트 | 파라미터 |
|---|---|
| `search_performed` | `query`, `result_count` |
| `book_added` | `source = "search"` |

---

## TODO

- [ ] `SearchScreen` Composable 구현 (Dark / Light 공통)
- [ ] 검색 입력 필드 구현 (`TextField`, 포커스 시 키보드 자동 올라오기)
- [ ] 검색 결과 리스트 아이템 컴포넌트
    - [ ] 책 표지 이미지 (Coil KMP)
    - [ ] 책 제목 / 저자 / 출판사 / 출판 연도
    - [ ] "Add to Library" 버튼
- [ ] 각 상태별 UI 구현 (Initial / Loading / Results / No Results / Error)
- [ ] No Results 상태: "수동으로 입력하기" 버튼 → Manual Book Entry 화면으로 이동
- [ ] `SearchViewModel` / MVI State, Action, SideEffect 정의
- [ ] Google Books API 검색 debounce 처리 (300ms)
- [ ] 도서 추가 완료 시 `BookRepository.upsertBook()` 연동
- [ ] 도서 추가 완료 후 Interstitial 광고 노출 (빈도 제한 로직 포함)
- [ ] Analytics: `search_performed`, `book_added(source=search)` 이벤트 로깅
- [ ] AdMob 배너 광고 영역 (화면 하단, 비프리미엄 사용자)
