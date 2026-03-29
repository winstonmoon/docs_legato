# 프로필 & 설정 (Profile & Settings)

[:fontawesome-solid-arrow-up-right-from-square: Stitch에서 보기](https://stitch.withgoogle.com/projects/9973353658769610659){ target="_blank" .md-button }

---

## UI 구성

> Stitch 디자인: **User Profile and Settings - Centered Stats** / **Profile & Settings (Light)**

### Stitch 스크린샷

=== "Dark"
    ![Profile Dark](https://lh3.googleusercontent.com/aida/ADBb0uhiuRXh3hrsiErIC43PymkOHJqi4hQKjgL4q-HGzVsuYKwRuH4eAjHdiwsaA61wVmlFVCXdYqaTvJVxUTQxig5misP9e3j-Vw_HMlU8RdOjR5BqVqYW_p0B0AcuqnrcDxxHFQ9gZkK4NOVXqaLuIysgyl_5xlEt4y82C212lksIVdR09BUnbLomYv-IQLYq9nO2uAy57qD7d9cpuvfdErYJHF4yv_lozh9eAjJlpOYbn3p0o1KsIl-2JYM)

=== "Light"
    ![Profile Light](https://lh3.googleusercontent.com/aida/ADBb0uidqx8vXUkB2QxnkOjrc3WQR-OjUWu37ZIIiSQysnr_GPLKcsXfM2pP3a7kohKIPrIfR99gDTwcaDCcKbLrUQr5U6eIPu7hTpCqigUwjsltko--38TAnRVjNnH0h8ZTt8FmkEswEIFqP24kLMOvUy2IiJE3ts2rvtFQeb11zPV5F5fISzcdiXfyFPGcpGAll569FSiXjeRtOFCUZ1OahOkb8uanCQcPee3vC6uy7A2rq49vz6rthVvu2Q)

### 프로필 섹션

| 구성 요소 | 설명 |
|---|---|
| 프로필 사진 | Coil KMP 로드, 기본 아바타 fallback, 편집 버튼(우측 하단 fab) |
| 사용자 이름 | Supabase Auth display_name |
| 부제 텍스트 | "Reading X books this year" |
| **독서 통계 3열** | **Books Read** / **Pages Read** / **Day Streak** (중앙 정렬, 구분선 포함) |
| AdMob 배너 | 화면 최하단 고정 (Sticky) |

### 독서 통계 레이아웃 (3열)

| 통계 | 설명 |
|---|---|
| Books Read | 완독한 권 수 |
| Pages Read | 총 읽은 페이지 수 |
| Day Streak | 연속 독서 일 수 |

### 설정 섹션

| 설정 항목 | 아이콘 색상 | 설명 |
|---|---|---|
| Language | 파랑 (`translate`) | 한국어 / 영어 / 일본어 / 중국어 |
| Dark Mode | 보라 (`dark_mode`) | `ThemeMode` 토글, DataStore 영속화 |
| Notifications | 빨강 (`notifications`) | FCM 토픽 구독 / 해제 |
| Account Details | 초록 (`person`) | 계정 정보 수정 |
| Privacy & Security | 슬레이트 (`lock`) | 보안 설정 |
| Sign Out | (버튼) | Supabase Auth signOut → 로그인 화면 |

### 앱 정보

- 앱 버전 (`BuildConfig.VERSION_NAME`)
- 오픈소스 라이선스

---

## MVI 구조

```kotlin
data class ProfileUiState(
    val userName: String = "",
    val email: String = "",
    val avatarUrl: String? = null,
    val booksRead: Int = 0,
    val pagesRead: Int = 0,
    val dayStreak: Int = 0,          // 연속 독서 일 수
    val themeMode: ThemeMode = ThemeMode.SYSTEM,
    val notificationsEnabled: Boolean = false,
    val language: String = "ko",
    val isLoading: Boolean = false,
)

sealed interface ProfileAction {
    data class ThemeModeChange(val mode: ThemeMode) : ProfileAction
    data class LanguageChange(val tag: String) : ProfileAction
    data object LogoutClick : ProfileAction
    data object DeleteAccountClick : ProfileAction
    data object DeleteAccountConfirm : ProfileAction
}

sealed interface ProfileSideEffect {
    data object NavigateToLogin : ProfileSideEffect
    data object ShowDeleteConfirmDialog : ProfileSideEffect
}
```

---

## 테마 모드

```kotlin
enum class ThemeMode { SYSTEM, LIGHT, DARK }

// 앱 루트에서 적용
LegatoTheme(
    themeMode = themeMode,
    content = { /* ... */ }
)
```

---

## Repository 연동

```kotlin
// 테마 변경
preferenceRepository.setThemeMode(ThemeMode.DARK)

// 언어 변경
preferenceRepository.setLanguage("en")

// 로그아웃
supabaseClient.auth.signOut()
```

---

## TODO

- [x] `ProfileScreen` Composable 구현 (Dark / Light 공통)
- [ ] 프로필 섹션 구현
    - [ ] 프로필 사진 (Coil KMP, 기본 아바타 fallback, 편집 FAB)
    - [ ] 사용자 이름 / "Reading X books this year" 표시
    - [ ] **독서 통계 3열 (Books Read / Pages Read / Day Streak)** 중앙 정렬 레이아웃
- [ ] 설정 Preferences 섹션 구현
    - [ ] Language 항목 (파랑 아이콘 + 현재 언어 표시 + `arrow_forward_ios`)
    - [ ] Dark Mode 토글 (`PreferenceRepository.setThemeMode()`)
- [ ] 설정 Account 섹션 구현
    - [ ] Notifications 토글 (FCM 토픽 구독 / 해제)
    - [ ] Account Details 항목 (`arrow_forward_ios`)
    - [ ] Privacy & Security 항목 (`arrow_forward_ios`)
    - [ ] Sign Out 버튼 (빨강 border)
- [ ] AdMob 배너 하단 고정 (Sticky)
- [ ] 앱 버전 정보 표시 (`BuildConfig.VERSION_NAME`)
- [ ] `ProfileViewModel` / MVI State, Action, SideEffect 정의
- [ ] `StatsRepository.observeReadingStats()` → dayStreak 포함 연동
- [x] `PreferenceRepository` DataStore 연동 (테마, 언어 영속화)
- [ ] Supabase Auth — 로그아웃 연동
- [ ] Analytics: `screen_view` 이벤트 로깅

**중국어(zh) 다국어 리소스 추가**
- [ ] `composeResources/values-zh/strings.xml` 생성 (전체 문자열 중국어 번역)
- [ ] `AppPreferences.Language` 또는 언어 태그 상수에 `"zh"` 추가
- [ ] 언어 선택 UI에 中文 옵션 추가 (LanguageChange Action 연동)
