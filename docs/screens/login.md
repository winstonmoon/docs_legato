# 로그인 (Login Screen)

[:fontawesome-solid-arrow-up-right-from-square: Stitch에서 보기](https://stitch.withgoogle.com/projects/9973353658769610659?node-id=47ad34a9fd2243bf910b17b4bb1d0b2a){ target="_blank" .md-button }

---

## UI 구성

> Stitch 디자인: **Login Screen - Improved Guest Visibility (Dark)**

| 구성 요소 | 설명 |
|---|---|
| 배경 효과 | Radial gradient dot grid 패턴 + primary 색상 glow blur 장식 (좌상단 / 우하단) |
| 앱 아이콘 + 브랜드명 | `auto_stories` Material 아이콘(72px) + "Legato" 텍스트 |
| 환영 문구 | "Welcome back, reader!" |
| 서브 타이틀 | "Continue your literary journey where you left off." |
| Google 로그인 버튼 | `OutlinedButton` — SVG Google 로고 — Supabase Auth OAuth |
| Apple 로그인 버튼 | `OutlinedButton` — SVG Apple 로고 — iOS 전용 (expect/actual) |
| "or" 구분선 | `HorizontalDivider` + 중앙 텍스트 |
| Login with Email 버튼 | `Button` (Primary, filled) → `EmailLoginScreen`으로 이동 |
| 회원가입 링크 | "Don't have an account?" + **Sign Up** 텍스트 버튼 |
| Continue as Guest | primary 텍스트 컬러 + 밑줄(`TextDecoration.Underline`) — 게스트 세션으로 홈 진입 |

!!! note "Continue as Guest 시각적 개선"
    기존 무채색 `TextButton`에서 **primary 색상 + 밑줄** 스타일로 변경하여 게스트 로그인 옵션의 가시성을 개선.

---

## 이메일 로그인 화면 (Email Login Screen)

로그인 화면에서 **Login with Email** 버튼을 누르면 별도 `EmailLoginScreen`으로 이동한다.

| 구성 요소 | 설명 |
|---|---|
| 뒤로가기 버튼 | `TopAppBar` 네비게이션 아이콘 → Login 화면으로 복귀 |
| 이메일 입력 | `OutlinedTextField` (keyboardType = Email) |
| 비밀번호 입력 | `OutlinedTextField` + 비밀번호 가시성 토글 |
| Log In 버튼 | `Button` (Primary, filled) — 로그인 실행 |

---

## MVI 구조

```kotlin
// LoginScreen State
data class LoginUiState(
    val isLoading: Boolean = false,
    val error: String? = null,
)

// LoginScreen Action
sealed interface LoginAction {
    data object GoogleLoginClick : LoginAction
    data object AppleLoginClick : LoginAction
    data object EmailLoginClick : LoginAction       // → EmailLoginScreen 이동
    data object GuestLoginClick : LoginAction       // → 게스트 세션으로 Home 이동
    data object SignUpClick : LoginAction
}

// LoginScreen SideEffect
sealed interface LoginSideEffect {
    data object NavigateToEmailLogin : LoginSideEffect
    data object NavigateToHome : LoginSideEffect
    data object NavigateToSignUp : LoginSideEffect
    data class ShowError(val message: String) : LoginSideEffect
}

// EmailLoginScreen State
data class EmailLoginUiState(
    val email: String = "",
    val password: String = "",
    val emailError: String? = null,
    val passwordError: String? = null,
    val isLoading: Boolean = false,
)

// EmailLoginScreen Action
sealed interface EmailLoginAction {
    data class EmailChange(val value: String) : EmailLoginAction
    data class PasswordChange(val value: String) : EmailLoginAction
    data object LoginClick : EmailLoginAction
}

// EmailLoginScreen SideEffect
sealed interface EmailLoginSideEffect {
    data object NavigateToHome : EmailLoginSideEffect
    data class ShowError(val message: String) : EmailLoginSideEffect
}
```

---

## 네비게이션

```
Login with Email 클릭 → EmailLoginScreen
Google / Apple 로그인 성공 → Home (백스택 제거)
Continue as Guest → Home (백스택 제거, 게스트 세션)
Sign Up 클릭 → SignUpScreen
EmailLoginScreen: 로그인 성공 → Home (백스택 제거)
EmailLoginScreen: 실패 → 에러 Snackbar 표시
```

**AppRoute 추가:**
```kotlin
@Serializable data object EmailLogin : AppRoute
```

---

## Supabase Auth 연동

```kotlin
// Google OAuth
supabaseClient.auth.signInWith(Google)

// Apple OAuth (iOS)
supabaseClient.auth.signInWith(Apple)

// 이메일/비밀번호 (EmailLoginScreen)
supabaseClient.auth.signInWith(Email) {
    this.email = email
    this.password = password
}

// 게스트 (Anonymous Auth)
supabaseClient.auth.signInAnonymously()
```

앱 시작 시 세션 자동 복원:
```kotlin
val session = supabaseClient.auth.currentSessionOrNull()
if (session != null) navigateToHome() else navigateToLogin()
```

---

## TODO

### UI 구현 — LoginScreen
- [x] `LoginScreen` Composable 구현 (Dark / Light 공통)
- [ ] `auto_stories` 아이콘 + "Legato" 브랜드 헤더 구현
- [ ] 환영 문구 / 서브 타이틀 텍스트 구현
- [ ] Google 소셜 로그인 버튼 UI 구현
- [ ] Apple 소셜 로그인 버튼 UI 구현 (iOS 전용 expect/actual)
- [ ] "Login with Email" Primary 버튼 구현 → `EmailLoginScreen`으로 이동
- [ ] Continue as Guest — primary 텍스트 컬러 + 밑줄 스타일로 구현
- [x] `LoginViewModel` / MVI State, Action, SideEffect 정의
- [ ] 로그인 성공 → Home 네비게이션 (백스택 제거)
- [ ] 로딩 중 버튼 비활성화 처리

### UI 구현 — EmailLoginScreen
- [ ] `EmailLoginScreen` Composable 구현
- [ ] 이메일 + 비밀번호 입력 폼 UI 구현
- [ ] 비밀번호 가시성 토글 구현
- [ ] `EmailLoginViewModel` / MVI State, Action, SideEffect 정의
- [ ] 로그인 성공 → Home 네비게이션 (백스택 제거)
- [ ] 로그인 실패 시 에러 메시지 표시 (Snackbar)
- [ ] `AppRoute.EmailLogin` 라우트 추가

### Google 로그인 외부 API 연동
- [ ] Google Cloud Console에서 OAuth 2.0 클라이언트 ID 생성 (Android / iOS 각각)
- [ ] `build.gradle`에 `supabase-kt` Google Auth 의존성 추가 (`io.github.jan-tennert.supabase:auth-kt`)
- [ ] Android: `google-services.json` 다운로드 및 프로젝트에 추가
- [ ] Android: `GoogleAuthProvider` 설정 (Web Client ID 등록)
- [ ] iOS: `GoogleService-Info.plist` 다운로드 및 프로젝트에 추가
- [ ] iOS: URL Scheme 등록 (Info.plist에 reversed client ID 추가)
- [ ] Supabase Dashboard에서 Google OAuth provider 활성화 및 Client ID / Secret 등록
- [ ] Supabase Auth — Google OAuth 연동 (`supabaseClient.auth.signInWith(Google)`)

### Apple 로그인 외부 API 연동
- [ ] Apple Developer Portal에서 App ID에 "Sign in with Apple" Capability 활성화
- [ ] Apple Developer Portal에서 Services ID 생성 (Web 인증용)
- [ ] Apple Developer Portal에서 Key 생성 (Sign in with Apple 권한 포함) 및 `.p8` 파일 다운로드
- [ ] iOS: Xcode 프로젝트에 "Sign in with Apple" Entitlement 추가
- [ ] Supabase Dashboard에서 Apple OAuth provider 활성화 및 Services ID / Key ID / `.p8` 등록
- [ ] Supabase Auth — Apple OAuth 연동 (`supabaseClient.auth.signInWith(Apple)`) — iOS 전용 expect/actual

### 게스트 로그인
- [ ] Supabase Anonymous Auth 활성화 (Dashboard → Auth → Providers → Anonymous)
- [ ] `GuestLoginClick` Action 처리 — `supabaseClient.auth.signInAnonymously()`
- [ ] 게스트 세션 → Home 네비게이션 (백스택 제거)
- [ ] 게스트 계정 전환 UI (Profile 화면에서 정식 계정으로 업그레이드 유도)

### 공통 인증 로직
- [ ] Supabase Auth — 이메일/비밀번호 로그인 연동
- [ ] 앱 시작 시 세션 자동 복원 로직 (세션 있으면 홈으로 바로 이동)
