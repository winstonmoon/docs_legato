# 도서관 / 히스토리 (Library History)

[:fontawesome-solid-arrow-up-right-from-square: Stitch에서 보기](https://stitch.withgoogle.com/projects/9973353658769610659){ target="_blank" .md-button }

---

## UI 구성

| 구성 요소 | 설명 |
|---|---|
| "Library" 헤더 | 상단 타이틀 |
| 상태 필터 칩 | All / Reading / Completed / Want to Read |
| 정렬 옵션 | 최근 추가순 / 제목순 / 저자순 |
| 도서 목록 | 리스트 뷰 |
| — 책 표지 썸네일 | Coil KMP 로드 |
| — 책 제목 / 저자 | 텍스트 |
| — 독서 상태 배지 | 색상 구분 |
| — 진행률 바 | READING 상태에서만 표시 |
| Empty State | 필터 결과 없을 때 안내 UI |
| AdMob 배너 | 화면 하단 (비프리미엄 사용자) |

---

## 독서 상태 (ReadingStatus)

| 상태 | 배지 색상 | 설명 |
|---|---|---|
| `READING` | 파랑 `#137FEC` | 읽는 중 |
| `FINISHED` | 초록 `#4CAF50` | 완독 |
| `WISHLIST` | 주황 `#FF9800` | 읽고 싶은 |
| `PAUSED` | 회색 `#9E9E9E` | 일시 중단 |
| `DROPPED` | 빨강 `#F44336` | 포기 |

---

## MVI 구조

```kotlin
data class LibraryUiState(
    val books: List<BookItemUiModel> = emptyList(),
    val selectedFilter: ReadingStatus? = null, // null = All
    val sortOrder: LibrarySortOrder = LibrarySortOrder.RECENTLY_ADDED,
    val isLoading: Boolean = false,
)

sealed interface LibraryAction {
    data class FilterChange(val status: ReadingStatus?) : LibraryAction
    data class SortChange(val order: LibrarySortOrder) : LibraryAction
    data class BookClick(val bookId: String) : LibraryAction
}

sealed interface LibrarySideEffect {
    data class NavigateToRecordProgress(val bookId: String) : LibrarySideEffect
}

enum class LibrarySortOrder { RECENTLY_ADDED, TITLE, AUTHOR }
```

---

## Repository 연동

```kotlin
bookRepository.observeBooks(status = selectedFilter)
    .combine(sortOrderFlow) { books, order ->
        books.sortedWith(order.comparator)
    }
```

---

## TODO

- [ ] `LibraryScreen` Composable 구현 (Dark / Light 공통)
- [ ] 상태 필터 칩 UI 구현 (Reading / Completed / Want to Read / All)
- [ ] 도서 목록 리스트 뷰 구현
    - [ ] 책 표지 썸네일 (Coil KMP)
    - [ ] 책 제목 / 저자
    - [ ] 독서 상태 배지 컴포넌트 (색상 구분)
    - [ ] 진행률 바 (READING 상태 전용)
- [ ] 정렬 옵션 UI (최근 추가순 / 제목순 / 저자순)
- [ ] 도서 카드 클릭 → Record Reading Progress 화면으로 이동
- [ ] `LibraryViewModel` / MVI State, Action, SideEffect 정의
- [ ] `BookRepository.observeBooks(status)` 연동 (상태별 필터)
- [ ] 도서 목록 없을 때 Empty State UI
- [ ] AdMob 배너 광고 영역 (화면 하단, 비프리미엄 사용자)
- [ ] Analytics: `screen_view` 이벤트 로깅
