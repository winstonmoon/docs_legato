# 홈 대시보드 (Home Dashboard)

[:fontawesome-solid-arrow-up-right-from-square: Stitch에서 보기](https://stitch.withgoogle.com/projects/9973353658769610659){ target="_blank" .md-button }

---

## UI 구성

| 구성 요소 | 설명 |
|---|---|
| 환영 메시지 | "안녕하세요, {username}" |
| 현재 읽는 책 카드 | 표지 이미지, 제목/저자, 진행률 바, "계속 읽기" 버튼 |
| 현재 읽는 책 없음 | Empty State UI |
| 최근 독서 활동 | 최근 독서 세션 목록 (날짜, 읽은 페이지) |
| 주간 통계 요약 | 이번 주 읽은 시간 / 페이지 수 카드 |
| Bottom Navigation | Home / Library / Search / Profile |

---

## MVI 구조

```kotlin
data class HomeUiState(
    val userName: String = "",
    val currentBook: BookItemUiModel? = null,
    val recentSessions: List<ReadingSessionUiModel> = emptyList(),
    val weeklyStats: WeeklyStatsUiModel? = null,
    val isLoading: Boolean = false,
)

sealed interface HomeAction {
    data object ContinueReadingClick : HomeAction
    data class SessionItemClick(val sessionId: String) : HomeAction
}

sealed interface HomeSideEffect {
    data class NavigateToRecordProgress(val bookId: String) : HomeSideEffect
}
```

---

## Repository 연동

```kotlin
// 현재 읽는 책
bookRepository.observeBooks(status = ReadingStatus.READING)

// 최근 세션
readingSessionRepository.observeRecentSessions(limit = 5)

// 주간 통계
statsRepository.observeWeeklyStats()
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

- [ ] `HomeScreen` Composable 구현 (Dark / Light 공통)
- [ ] 상단 환영 메시지 + 사용자 이름 표시 (Supabase Auth user.displayName 연동)
- [ ] 현재 읽는 책 카드 컴포넌트 구현
    - [ ] 책 표지 이미지 (Coil KMP로 로드)
    - [ ] 책 제목 / 저자 텍스트
    - [ ] 진행률 `LinearProgressIndicator` (현재 페이지 / 전체 페이지)
    - [ ] "계속 읽기" 버튼 → Record Reading Progress 화면으로 이동
- [ ] 현재 읽는 책 없을 때 Empty State UI
- [ ] 최근 독서 세션 목록 (날짜, 읽은 페이지 수)
- [ ] 주간 독서 통계 요약 카드 (읽은 시간 / 페이지 수)
- [ ] `HomeViewModel` / MVI State, Action, SideEffect 정의
- [ ] `ReadingSessionRepository.observeRecentSessions()` 연동
- [ ] `StatsRepository.observeWeeklyStats()` 연동
- [ ] Bottom Navigation Bar 구현 (Home / Library / Search / Profile)
- [ ] Android: `WindowWidthSizeClass` 기반 Adaptive Navigation 구현
