# 홈 대시보드 (Home Dashboard)

[:fontawesome-solid-arrow-up-right-from-square: Stitch에서 보기](https://stitch.withgoogle.com/projects/9973353658769610659){ target="_blank" .md-button }

---

## UI 구성

> Stitch 디자인: **Home Dashboard - Updated Stats Layout v2** / **Home Dashboard (Light)**

### Stitch 스크린샷

=== "Dark"
    ![Home Dashboard Dark](https://lh3.googleusercontent.com/aida/ADBb0ujA69r2o8-huWkfE_pjGP1zccyN1ym1AfdYCbcVprD_UcOVt6Ir5ArW-dH3aokUg8wCamuh4XxbFfo1vC-CrY3yrTQZvJ1O70z4_BuMsycdfwbrz63TYLCkN-ekoeaILyArXmDVM16KL_D-UD0njPXQoQadJNCNP_XGAR0B2HiZL2eVaA6b1pxwPCGr1BvLYM2soC7fuKpXi0csU0_k8yokfackaTYNqgQE663wQX_aQC3z9esGCr_grtM)

=== "Light"
    ![Home Dashboard Light](https://lh3.googleusercontent.com/aida/ADBb0uge749hhYmHJdwg5opEQWN8sh9i88uFazt2FeB6jBjodRvfzYYAvRM66_XGkMTZnxVf6abqY8XVILVf1zzFdGbVaBLCnR1HwOrM30xDuefZQQYP-sfZn6zBsbArhKj-HXEVn0CpKTu64GusxqzWm0JwmEytJrlvTFkLli0jPr8BwE58OT7svYUqBZKsIS6uebezLJvx9RW43Yyq30F7ysVbraQd_SrI_MBlc47xMO13pBVlrWjb0qnYPw)

| 구성 요소 | 설명 |
|---|---|
| Current Read 카드 | 표지 이미지, 제목/저자, Chapter X of Y, 진행률 바(%), "Continue Reading" 버튼 |
| 현재 읽는 책 없음 | Empty State UI |
| Reading Stats | 3열 카드 그리드: **Books Read** / **Pages Read** / **Day Streak** |
| From Your Library | 최근 추가된 도서 가로 스크롤 썸네일 목록 |
| AdMob 배너 | 하단 네비게이션 바 바로 위 고정 배너 (비프리미엄 사용자) |
| Bottom Navigation | Home · Search · Library · Profile |

### Reading Stats 카드 상세

| 통계 | 아이콘 | 설명 |
|---|---|---|
| Books Read | `menu_book` (파랑) | 완독한 권 수 |
| Pages Read | `description` (파랑) | 총 읽은 페이지 수 |
| Day Streak | `whatshot` (주황) | 연속 독서 일 수 |

---

## MVI 구조

```kotlin
data class ReadingStats(
    val booksRead: Int = 0,
    val pagesRead: Int = 0,
    val dayStreak: Int = 0,
)

data class HomeUiState(
    val userName: String = "",
    val currentBook: BookItemUiModel? = null,
    val readingStats: ReadingStats = ReadingStats(),
    val libraryPreview: List<BookItemUiModel> = emptyList(),  // "From Your Library" 최근 도서
    val isLoading: Boolean = false,
)

sealed interface HomeAction {
    data object ContinueReadingClick : HomeAction
    data class LibraryBookClick(val bookId: String) : HomeAction
    data object ViewAllClick : HomeAction
}

sealed interface HomeSideEffect {
    data class NavigateToRecordProgress(val bookId: String) : HomeSideEffect
    data object NavigateToLibrary : HomeSideEffect
}
```

---

## Repository 연동

```kotlin
// 현재 읽는 책
bookRepository.observeBooks(status = ReadingStatus.READING)

// Reading Stats (완독 수, 총 페이지, 연속 독서일)
statsRepository.observeReadingStats()

// From Your Library (최근 추가된 도서 미리보기)
bookRepository.observeRecentBooks(limit = 5)
```

---

## Android Adaptive Navigation

| 화면 크기 | 네비게이션 형태 |
|---|---|
| `Compact` (< 600dp) | Bottom Navigation Bar |
| `Medium` (600–840dp) | Navigation Rail |
| `Expanded` (> 840dp) | Navigation Drawer |

```kotlin
val windowSizeClass = calculateWindowSizeClass()
when (windowSizeClass.widthSizeClass) {
    WindowWidthSizeClass.Compact -> BottomBar()
    WindowWidthSizeClass.Medium -> NavigationRail()
    WindowWidthSizeClass.Expanded -> NavigationDrawer()
}
```

---

## TODO

- [x] `HomeScreen` Composable 구현 (Dark / Light 공통)
- [x] Current Read 카드 컴포넌트 구현
    - [x] 책 표지 이미지 (Coil KMP `AsyncImage` 로드)
    - [x] 책 제목 / 저자 텍스트
    - [ ] Chapter X of Y 표시
    - [x] 진행률 `LinearProgressIndicator` (%) + 퍼센트 텍스트
    - [x] "Continue Reading" 버튼 → Record Reading Progress 화면으로 이동
- [x] 현재 읽는 책 없을 때 Empty State UI
- [x] Reading Stats 3열 카드 그리드 구현
    - [x] Books Read (`menu_book`, 파랑 배경)
    - [x] Pages Read (`description`, 파랑 배경)
    - [x] Day Streak (`whatshot`, 주황 배경) — 현재 값 하드코딩 0, 실 계산 로직 미구현
- [x] "From Your Library" 가로 스크롤 섹션 구현 + "View All" 버튼
- [ ] AdMob 배너 광고 (하단 네비게이션 바 위에 고정, 비프리미엄 사용자)
- [x] `HomeViewModel` / MVI State, Action, SideEffect 정의
- [x] `LocalBookRepository.observeReadingStats()` 연동 (Books Read, Pages Read 연동; Day Streak은 0 하드코딩)
- [x] `LocalBookRepository.observeAllBooks().take(5)` 연동 (`observeRecentBooks()` 아닌 전체 목록 상위 5개)
- [x] Bottom Navigation Bar 구현 (Home · Search · Library · Profile)
- [ ] Android: `WindowWidthSizeClass` 기반 Adaptive Navigation 구현
