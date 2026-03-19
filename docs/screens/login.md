# 로그인 (Login Screen)

[:fontawesome-solid-arrow-up-right-from-square: Stitch에서 보기](https://stitch.withgoogle.com/projects/9973353658769610659){ target="_blank" .md-button }

---

## UI 구성

| 구성 요소 | 설명 |
|---|---|
| Smiling Book 로고 | `composeResources/drawable`에 SVG로 추가 |
| 앱 이름 & 슬로건 | "Legato — Connected Reading" |
| Google 로그인 버튼 | Supabase Auth OAuth |
| Apple 로그인 버튼 | iOS 전용 (expect/actual) |
| 이메일 로그인 폼 | 이메일 + 비밀번호 입력 |
| 회원가입 안내 | 하단 텍스트 링크 |

---

## MVI 구조

```kotlin
// State
data class LoginUiState(
    val isLoading: Boolean = false,
    val error: String? = null,
)

// Action
sealed interface LoginAction {
    data class GoogleLoginClick : LoginAction
    data class AppleLoginClick : LoginAction
    data class EmailLoginClick(val email: String, val password: String) : LoginAction
}

// SideEffect
sealed interface LoginSideEffect {
    data object NavigateToHome : LoginSideEffect
    data class ShowError(val message: String) : LoginSideEffect
}
```

---

## 네비게이션

```
로그인 성공 → Home (백스택 제거)
로그인 실패 → 에러 Snackbar 표시
```

---

## Supabase Auth 연동

```kotlin
// Google OAuth
supabaseClient.auth.signInWith(Google)

// Apple OAuth (iOS)
supabaseClient.auth.signInWith(Apple)

// 이메일/비밀번호
supabaseClient.auth.signInWith(Email) {
    this.email = email
    this.password = password
}
```

앱 시작 시 세션 자동 복원:
```kotlin
val session = supabaseClient.auth.currentSessionOrNull()
if (session != null) navigateToHome() else navigateToLogin()
```

---

## TODO

- [ ] `LoginScreen` Composable 구현 (Dark / Light 공통)
- [ ] Smiling Book SVG 로고 에셋 추가 (`composeResources/drawable`)
- [ ] Google 소셜 로그인 버튼 UI 구현
- [ ] Apple 소셜 로그인 버튼 UI 구현 (iOS 전용 expect/actual)
- [ ] 이메일 로그인 폼 UI 구현 (이메일 입력 + 비밀번호 입력)
- [ ] `LoginViewModel` / MVI State, Action, SideEffect 정의
- [ ] Supabase Auth — Google OAuth 연동
- [ ] Supabase Auth — Apple OAuth 연동
- [ ] Supabase Auth — 이메일/비밀번호 로그인 연동
- [ ] 앱 시작 시 세션 자동 복원 로직 (세션 있으면 홈으로 바로 이동)
- [ ] 로그인 성공 → Home 네비게이션 (백스택 제거)
- [ ] 로그인 실패 시 에러 메시지 표시 (Snackbar)
- [ ] 로딩 중 버튼 비활성화 처리
