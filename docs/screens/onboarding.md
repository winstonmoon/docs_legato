# 온보딩 (Onboarding: Personalization)

[:fontawesome-solid-arrow-up-right-from-square: Stitch에서 보기](https://stitch.withgoogle.com/projects/9973353658769610659?node-id=907554791dde468194fe221102fde4ee){ target="_blank" .md-button }

---

## UI 구성

| 구성 요소 | 설명 |
|---|---|
| "Set Your Reading Path" 타이틀 | 화면 상단 |
| 월간 독서 목표 슬라이더 | 1~15권 범위, 단계 레이블: Casual / Avid / Bibliophile |
| 장르 선택 그리드 | Fiction, Sci-Fi, Mystery, History, Philosophy, Biography 등, 1~3개 복수 선택 |
| Get Started 버튼 | `Button` (Primary, filled) — 온보딩 완료 후 Home으로 이동 |
| 건너뛰기 버튼 | `TextButton` — 온보딩 생략 후 Home으로 이동 |

### 월간 독서 목표 단계

| 범위 | 레이블 |
|---|---|
| 1~4권 | Casual |
| 5~9권 | Avid |
| 10~15권 | Bibliophile |

### 선택 가능 장르

`Fiction`, `Sci-Fi`, `Mystery`, `History`, `Philosophy`, `Biography`, `Self-Help`, `Technology`
(1~3개 선택, 미선택도 허용)

---

## MVI 구조

```kotlin
// State
data class OnboardingUiState(
    val monthlyGoal: Int = 5,
    val selectedGenres: Set<String> = emptySet(),
    val isLoading: Boolean = false,
)

// Action
sealed interface OnboardingAction {
    data class MonthlyGoalChanged(val value: Int) : OnboardingAction
    data class GenreToggled(val genre: String) : OnboardingAction
    data object GetStartedClick : OnboardingAction
    data object SkipClick : OnboardingAction
}

// SideEffect
sealed interface OnboardingSideEffect {
    data object NavigateToHome : OnboardingSideEffect
}
```

---

## 네비게이션

```
SignUpScreen 성공 → OnboardingScreen (백스택 제거)
Get Started → Home (백스택 제거)
건너뛰기 → Home (백스택 제거)
```

**AppRoute 추가:**
```kotlin
@Serializable data object Onboarding : AppRoute
```

---

## Supabase 연동

```kotlin
// 온보딩 데이터 저장 (profiles 테이블 upsert)
supabaseClient.from("profiles").upsert(
    mapOf(
        "id" to supabaseClient.auth.currentUserOrNull()?.id,
        "monthly_goal" to monthlyGoal,
        "genre_prefs" to selectedGenres.toList(),
    )
)
```

---

## TODO

### UI 구현
- [ ] `OnboardingScreen` Composable 구현
- [ ] 월간 목표 슬라이더 (1~15, Casual / Avid / Bibliophile 단계 레이블 표시)
- [ ] 장르 선택 그리드 (멀티 선택 칩, 1~3개 제한)
- [ ] Get Started `Button` 구현
- [ ] 건너뛰기 `TextButton` 구현
- [ ] `OnboardingViewModel` / MVI State, Action, SideEffect 정의
- [ ] `AppRoute.Onboarding` 라우트 추가

### Supabase 연동
- [ ] `profiles` 테이블 `monthly_goal`, `genre_prefs` 컬럼 upsert
- [ ] 건너뛰기 시 저장 생략 후 Home 이동
