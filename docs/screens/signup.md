# 회원가입 (Sign Up Screen)

[:fontawesome-solid-arrow-up-right-from-square: Stitch에서 보기](https://stitch.withgoogle.com/projects/9973353658769610659?node-id=1c644d6367e24692ab1e2d0d776527ce){ target="_blank" .md-button }

---

## UI 구성

| 구성 요소 | 설명 |
|---|---|
| 뒤로가기 버튼 | `TopAppBar` 네비게이션 아이콘 → Login 화면으로 복귀 |
| "Create Account" 타이틀 | 화면 상단 |
| 닉네임 입력 | `OutlinedTextField` — 프로필 표시용 |
| 이메일 입력 | `OutlinedTextField` (keyboardType = Email) |
| 비밀번호 입력 | `OutlinedTextField` + 가시성 토글 |
| 출생연도 입력 | `OutlinedTextField` (keyboardType = Number) |
| 직업군 선택 | `ExposedDropdownMenu` — 학생 / 직장인 / 기타 |
| Sign Up 버튼 | `Button` (Primary, filled) |
| 로그인 링크 | "Already have an account?" + **Log In** 텍스트 버튼 |

---

## 설계 방향

회원가입 1화면에서 **핵심 프로필 데이터를 함께 수집**한다.
출생연도와 직업군은 선택(optional) 항목이지만 화면에 노출하여 AdMob CPM 최적화와 구독 가격 A/B 테스트에 활용한다.

가입 완료 후에는 **온보딩 플로우**로 이동하여 독서 목표 및 선호 장르를 추가로 수집한다.

> **참고**: Google / Apple 소셜 로그인으로 가입 시 이메일·비밀번호 입력 없이 가입 가능.
> 소셜 로그인 우선 유도 → 이메일 가입을 폴백으로 제공하는 구조 권장.

---

## 수집 항목

### 필수 (회원가입 1화면)

| 항목 | 용도 |
|---|---|
| 닉네임 | 프로필 표시, 추후 소셜 기능 대비 |
| 이메일 | 로그인 식별자, 비밀번호 재설정 |
| 비밀번호 | 인증 (소셜 로그인 시 불필요) |
| 출생연도 | AdMob 연령대별 CPM 최적화 (선택 입력) |
| 직업군 | 구독 가격 A/B 테스트 활용 (선택 입력) |

### 선택 (온보딩 플로우, 가입 직후)

| 항목 | 수집 방식 | 수익화 연결 |
|---|---|---|
| **독서 목표** (월 몇 권) | 슬라이더 | 프리미엄 플랜 제안 타이밍 |
| **선호 장르** (1~3개 선택) | 멀티 칩 선택 | 도서 추천 광고, 제휴 판매 타겟팅 |
| **독서 알림 선호** | ON/OFF 토글 | FCM 기반 리텐션 마케팅 |

---

## 화면 플로우

```
[회원가입]
  닉네임 + 이메일 + 비밀번호 + 출생연도(선택) + 직업군(선택)
     │
     └──▶ [온보딩: Personalization]
               독서 목표 슬라이더 + 선호 장르 선택
               │
               └──▶ Home
```

- 온보딩에는 "건너뛰기" 버튼 제공 (강제하지 않음)
- 건너뛴 항목은 Profile & Settings에서 재설정 가능

---

## MVI 구조

```kotlin
// State
data class SignUpUiState(
    val nickname: String = "",
    val email: String = "",
    val password: String = "",
    val birthYear: String = "",
    val occupation: String = "",           // "student" | "worker" | "other" | ""
    val nicknameError: String? = null,
    val emailError: String? = null,
    val passwordError: String? = null,
    val isLoading: Boolean = false,
    val error: String? = null,
) {
    val isSubmitEnabled: Boolean
        get() = nickname.isNotBlank() && email.isNotBlank() && password.isNotBlank()
}

// Action
sealed interface SignUpAction {
    data class NicknameChanged(val value: String) : SignUpAction
    data class EmailChanged(val value: String) : SignUpAction
    data class PasswordChanged(val value: String) : SignUpAction
    data class BirthYearChanged(val value: String) : SignUpAction
    data class OccupationChanged(val value: String) : SignUpAction
    data object SignUpClick : SignUpAction
    data object GoogleSignUpClick : SignUpAction
    data object AppleSignUpClick : SignUpAction
    data object LoginClick : SignUpAction
}

// SideEffect
sealed interface SignUpSideEffect {
    data object NavigateToOnboarding : SignUpSideEffect
    data object NavigateToLogin : SignUpSideEffect
    data class ShowError(val message: String) : SignUpSideEffect
}
```

---

## Supabase Auth 연동

```kotlin
// 이메일/비밀번호 회원가입
supabaseClient.auth.signUpWith(Email) {
    this.email = email
    this.password = password
    data = buildJsonObject {
        put("nickname", nickname)
        if (birthYear.isNotBlank()) put("birth_year", birthYear.toInt())
        if (occupation.isNotBlank()) put("job_type", occupation)
    }
}

// Google OAuth (가입 & 로그인 통합)
supabaseClient.auth.signInWith(Google)

// Apple OAuth (iOS 전용)
supabaseClient.auth.signInWith(Apple)
```

가입 완료 후 `user_metadata`에 닉네임 저장. 온보딩 데이터는 별도 `profiles` 테이블에 upsert.

---

## DB 스키마 (profiles 테이블)

```sql
create table profiles (
  id          uuid references auth.users on delete cascade primary key,
  nickname    text not null,
  birth_year  smallint,           -- 선택 수집 (회원가입 화면)
  job_type    text,               -- 'student' | 'worker' | 'other' (회원가입 화면)
  genre_prefs text[],             -- 선호 장르 배열 (온보딩)
  monthly_goal smallint,          -- 월 독서 목표 권수 (온보딩)
  notify_enabled boolean default true,
  created_at  timestamptz default now()
);
```

---

## 네비게이션

```
회원가입 성공 → OnboardingScreen (백스택 제거)
회원가입 실패 → 에러 Snackbar 표시
로그인 화면으로 이동 → LoginScreen (텍스트 링크)
```

---

## TODO

### UI 구현
- [ ] `SignUpScreen` Composable 구현 (Light / Dark)
- [ ] 닉네임 + 이메일 + 비밀번호 입력 폼
- [ ] 출생연도 숫자 입력 필드 (keyboardType = Number, 선택)
- [ ] 직업군 드롭다운 선택 UI (학생 / 직장인 / 기타, 선택)
- [ ] 비밀번호 가시성 토글
- [ ] Google / Apple 소셜 가입 버튼
- [ ] `SignUpViewModel` / MVI State, Action, SideEffect 정의
- [ ] 이메일 유효성 / 비밀번호 최소 길이 검사
- [ ] 가입 성공 → OnboardingScreen 네비게이션
- [ ] 이메일 중복 에러 처리 (Snackbar)

### 온보딩 플로우
- [ ] `OnboardingScreen` Composable 구현 (→ [온보딩 화면 스펙](onboarding.md) 참고)

### Supabase 연동
- [ ] `profiles` 테이블 생성 (위 스키마 참고)
- [ ] Row Level Security (RLS) 정책: 본인 row만 read/write 허용
- [ ] 이메일/비밀번호 회원가입 연동
- [ ] 소셜 가입 후 `profiles` 자동 생성 (DB Function or Edge Function 활용)
