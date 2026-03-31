# 도서관 / 히스토리 (Library History)

[:fontawesome-solid-arrow-up-right-from-square: Stitch에서 보기](https://stitch.withgoogle.com/projects/9973353658769610659){ target="_blank" .md-button }

---

## UI 구성

> Stitch 디자인: **Library History - Navigation Updated** / **Library History (Light)**

### Stitch 스크린샷

=== "Dark"
    ![Library History Dark](https://lh3.googleusercontent.com/aida/ADBb0uhM_iwQMVcniI3XgT_N-GhtKOIFKY0qtQ4DDggjYM9OpzF81ncu63anHrwtmNTeFS95Q09yOrG5NQMsnEFKgInBV2BMuJ1dUM4xlYH_SyA0r2vUjwHvNgbRm16hs1pYdjTJ0cE0ukpo3i7vDLiXLEAmqhAvS5KEZSy6yqL0gUcfIyN5bpTLbLbD0GVzlYMN8zJNo4XTtZE8Kj3B1EUaZg6IsHZOcNU1XqZIuF7j4WsaxsSTVac8blh19Zs)

=== "Light"
    ![Library History Light](https://lh3.googleusercontent.com/aida/ADBb0uh3BdPZS3fnCNljMRJT-eI_ECenZWOTyECrVT_OpWLHedjq68ym-2pUck9kQnUpPX1FmEG7vyvuD3iXgTi6usaK7782O79rIHR2dfjTVjM9mXd6iEld09Jc9JG2317j3EIvGmfsPIj9dgKGBk9r1RTsqtXFCXXhBz4PTjd58TYTX0hUmvLsQbC5uAc5hHlw_0z107v3dNRHl-XwOvytI0qBwCHjVMjRUfgZeAQQy7yV47KWa93oNrIJuYU)

| 구성 요소 | 설명 |
|---|---|
| **월별 섹션 헤더** | "September 2023" 형식 + 해당 월 도서 수 배지 (예: "3 Books") |
| 도서 그리드 | **2열 그리드** (aspect-ratio 2:3 표지) |
| — 책 표지 썸네일 | Coil KMP 로드, hover 시 scale 효과 |
| — 완독 배지 | `DONE` (초록 pill 배지, `ReadingStatus.FINISHED`) |
| — 책 제목 | truncate 처리 |
| — 완료일 + 별점 | "Sep 2 · ★ 5.0" 형식 |
| Empty State | 필터 결과 없을 때 안내 UI |
| Bottom Navigation | Library 탭 활성화 시 pill 배경 강조 (`bg-primary/10`) |
| AdMob 배너 | 화면 하단 (비프리미엄 사용자) |

!!! note "히스토리 뷰 구조"
    Library 화면은 **월별 그룹화** 히스토리 뷰가 핵심.
    완독 날짜 기준으로 `YYYY년 MM월` 섹션으로 묶어 내림차순 정렬.

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
data class LibraryMonthGroup(
    val year: Int,
    val month: Int,
    val books: List<BookItemUiModel>,
)

data class LibraryUiState(
    val monthGroups: List<LibraryMonthGroup> = emptyList(),  // 월별 그룹화 히스토리
    val isLoading: Boolean = false,
)

sealed interface LibraryAction {
    data class BookClick(val bookId: String) : LibraryAction
}

sealed interface LibrarySideEffect {
    data class NavigateToRecordProgress(val bookId: String) : LibrarySideEffect
}
```

---

## Repository 연동

```kotlin
// 완독 도서를 월별 그룹으로 변환
bookRepository.observeBooks(status = ReadingStatus.FINISHED)
    .map { books ->
        books
            .sortedByDescending { it.finishedDate }
            .groupBy { YearMonth(it.finishedDate.year, it.finishedDate.monthNumber) }
            .map { (ym, books) -> LibraryMonthGroup(ym.year, ym.month, books) }
    }
```

---

## TODO

- [x] `LibraryScreen` Composable 구현 (Dark / Light 공통)
- [x] 월별 섹션 헤더 구현 ("MMMM YYYY" 형식 + 책 수 배지)
- [x] 도서 2열 그리드 구현
    - [x] 책 표지 썸네일 (Coil KMP `AsyncImage`, aspect-ratio 2:3)
    - [ ] hover/press 시 scale 애니메이션
    - [x] `DONE` 완독 배지 (초록 pill, 우측 상단 오버레이) — `DoneBadge` 컴포넌트 구현됨
    - [x] 책 제목 (truncate) + 완료일 + 별점
- [ ] Library 탭 활성화 시 pill 배경 강조 네비게이션 구현
- [ ] 도서 카드 클릭 → **Record Reading Progress** 화면으로 이동 (현재 구현은 BookDetail — RecordProgress로 수정 필요)
- [x] `LibraryViewModel` / MVI State, Action, SideEffect 정의
- [x] `LocalBookRepository.observeFinishedBooks()` → 월별 그룹화 변환 로직 (`toMonthLabel()`)
- [x] 도서 없을 때 Empty State UI — `LibraryEmptyState` 컴포넌트 구현됨
- [ ] AdMob 배너 광고 영역 (화면 하단, 비프리미엄 사용자)
- [ ] Analytics: `screen_view` 이벤트 로깅
