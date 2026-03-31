# 도서 상세 (Book Detail)

[:fontawesome-solid-arrow-up-right-from-square: Stitch에서 보기](https://stitch.withgoogle.com/projects/9973353658769610659){ target="_blank" .md-button }

---

## UI 구성

| 구성 요소 | 설명 |
|---|---|
| 뒤로가기 버튼 | `TopAppBar` 네비게이션 아이콘 |
| 책 표지 | 상단 전체 너비 이미지 (Coil KMP `AsyncImage`) |
| 책 제목 | 중앙 정렬 텍스트 |
| 저자 | 제목 아래 보조 텍스트 |
| 독서 상태 셀렉터 | `READING` / `FINISHED` / `WISHLIST` / `PAUSED` / `DROPPED` 탭 선택 |
| 진행률 바 | `LinearProgressIndicator` (현재 페이지 / 전체 페이지) |
| 별점 | 1~5점 StarRating 컴포넌트 |
| Log Reading 버튼 | 독서 기록 화면(`LogReadingScreen`)으로 이동 |
| Record Progress 버튼 | 독서 진행 기록 화면(`RecordProgressScreen`)으로 이동 |
| 삭제 버튼 | 도서 삭제 — `AlertDialog` 확인 후 실행 |

---

## MVI 구조

```kotlin
data class BookDetailUiState(
    val book: BookItemUiModel? = null,
    val isLoading: Boolean = false,
)

sealed interface BookDetailAction {
    data class StatusChange(val status: ReadingStatus) : BookDetailAction
    data class RatingChange(val rating: Int) : BookDetailAction
    data object LogReadingClick : BookDetailAction
    data object RecordProgressClick : BookDetailAction
    data object DeleteClick : BookDetailAction
    data object DeleteConfirm : BookDetailAction
}

sealed interface BookDetailSideEffect {
    data object NavigateBack : BookDetailSideEffect
    data object NavigateToLogReading : BookDetailSideEffect
    data object NavigateToRecordProgress : BookDetailSideEffect
    data object ShowDeleteConfirmDialog : BookDetailSideEffect
}
```

---

## 네비게이션

```
Library 도서 카드 클릭 → BookDetailScreen
Log Reading 버튼 → LogReadingScreen
Record Progress 버튼 → RecordProgressScreen
삭제 확인 → 뒤로가기 (Library)
```

---

## TODO

- [ ] `BookDetailScreen` Composable 구현
- [ ] 책 표지 / 제목 / 저자 헤더 구현
- [ ] 독서 상태 셀렉터 UI (5종 탭)
- [ ] 진행률 `LinearProgressIndicator`
- [ ] 별점 1~5 `StarRating` 컴포넌트
- [ ] Log Reading / Record Progress 이동 버튼
- [ ] 삭제 `AlertDialog` 구현
- [ ] `BookDetailViewModel` / MVI State, Action, SideEffect 정의
- [ ] `BookRepository.observeBook(id)` 연동
- [ ] `BookRepository.updateStatus()` / `updateRating()` 연동
- [ ] `AppRoute.BookDetail` 라우트 추가
