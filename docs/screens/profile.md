# 프로필 & 설정 (Profile & Settings)

[:fontawesome-solid-arrow-up-right-from-square: Stitch에서 보기](https://stitch.withgoogle.com/projects/9973353658769610659){ target="_blank" .md-button }

---

## UI 구성

### 프로필 섹션

| 구성 요소 | 설명 |
|---|---|
| 프로필 사진 | Coil KMP 로드, 기본 아바타 fallback |
| 사용자 이름 | Supabase Auth display_name |
| 이메일 | Supabase Auth email |
| 독서 통계 요약 | 완독 권수, 총 읽은 페이지 |

### 설정 섹션

| 설정 항목 | 설명 |
|---|---|
| 다크 / 라이트 모드 | `ThemeMode` 토글, DataStore 영속화 |
| 알림 설정 | FCM 토픽 구독 / 해제 |
| 언어 설정 | 한국어 / 영어 / 일본어 |
| 로그아웃 | Supabase Auth signOut → 로그인 화면 |
| 계정 삭제 | 확인 다이얼로그 → Supabase Auth deleteUser |

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
    val totalBooksCompleted: Int = 0,
    val totalPagesRead: Int = 0,
    val themeMode: ThemeMode = ThemeMode.SYSTEM,
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

- [ ] `ProfileScreen` Composable 구현 (Dark / Light 공통)
- [ ] 프로필 섹션 구현
    - [ ] 프로필 사진 (Coil KMP, 기본 아바타 fallback)
    - [ ] 사용자 이름 / 이메일 표시
    - [ ] 독서 통계 요약 카드 (완독 권수, 총 읽은 페이지)
- [ ] 설정 섹션 구현
    - [ ] 다크 / 라이트 모드 토글 (`PreferenceRepository.setThemeMode()`)
    - [ ] 알림 설정 (FCM 토픽 구독 / 해제)
    - [ ] 언어 설정 (`PreferenceRepository.setLanguage()`)
    - [ ] 로그아웃 버튼 (Supabase Auth signOut → Login 화면으로 이동)
    - [ ] 계정 삭제 버튼 (확인 다이얼로그 포함)
- [ ] 앱 버전 정보 표시
- [ ] `ProfileViewModel` / MVI State, Action, SideEffect 정의
- [ ] `PreferenceRepository` DataStore 연동 (테마, 언어 영속화)
- [ ] Supabase Auth — 로그아웃 연동
- [ ] Analytics: `screen_view` 이벤트 로깅
