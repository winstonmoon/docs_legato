# 회원가입 (Sign Up Screen)

> Stitch 디자인 화면 미작성 — 추후 구현 시 추가 예정

---

## 설계 방향

회원가입 자체는 **최소 필드**로 유지하고, 수익화에 필요한 데이터는 **가입 직후 온보딩 플로우**에서 선택적으로 수집한다.

---

## 수집 항목

### 필수 (회원가입 1화면)

| 항목 | 용도 |
|---|---|
| 이메일 | 로그인 식별자, 비밀번호 재설정 |
| 비밀번호 | 인증 (소셜 로그인 시 불필요) |
| 닉네임 | 프로필 표시, 추후 소셜 기능 대비 |

> **참고**: Google / Apple 소셜 로그인을 지원하면 이메일·비밀번호 입력 없이 가입 가능.
> 소셜 로그인 우선 유도 → 이메일 가입을 폴백으로 제공하는 구조 권장.

---

### 선택 (온보딩 플로우, 가입 직후 2~3단계)

| 항목 | 수집 방식 | 수익화 연결 |
|---|---|---|
| **독서 목표** (월 몇 권) | 슬라이더 or 버튼 선택 | 프리미엄 플랜 제안 타이밍 |
| **선호 장르** (1~3개 선택) | 멀티 칩 선택 | 도서 추천 광고, 제휴 판매 타겟팅 |
| **독서 알림 선호** | ON/OFF 토글 | FCM 기반 리텐션 마케팅 |
| **출생연도** | 년도 드롭다운 (선택) | 연령대별 AdMob 광고 단가(CPM) 최적화 |
| **직업군** | 학생 / 직장인 / 기타 | 프리미엄 가격 탄력적 적용 |

> **우선순위**: 출생연도 하나만 추가한다면 AdMob CPM 향상 효과가 가장 큼.
> 직업군은 추후 구독 가격 A/B 테스트 시 활용 가능.

---

## 화면 플로우

```
[회원가입] ──── 이메일 + 비밀번호 + 닉네임
     │
     └──▶ [온보딩 Step 1] 독서 목표 설정
               │
               └──▶ [온보딩 Step 2] 선호 장르 선택
                         │
                         └──▶ [온보딩 Step 3] 알림 & 출생연도 (선택)
                                   │
                                   └──▶ Home
```

- 온보딩은 각 단계에 "건너뛰기" 버튼 제공 (강제하지 않음)
- 건너뛴 항목은 Profile & Settings에서 언제든 재설정 가능

---

## MVI 구조

```kotlin
// State
data class SignUpUiState(
    val email: String = "",
    val password: String = "",
    val nickname: String = "",
    val isLoading: Boolean = false,
    val error: String? = null,
)

// Action
sealed interface SignUpAction {
    data class EmailChanged(val email: String) : SignUpAction
    data class PasswordChanged(val password: String) : SignUpAction
    data class NicknameChanged(val nickname: String) : SignUpAction
    data object SignUpClick : SignUpAction
    data object GoogleSignUpClick : SignUpAction
    data object AppleSignUpClick : SignUpAction
}

// SideEffect
sealed interface SignUpSideEffect {
    data object NavigateToOnboarding : SignUpSideEffect
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
  birth_year  smallint,           -- 선택 수집
  job_type    text,               -- 'student' | 'worker' | 'other'
  genre_prefs text[],             -- 선호 장르 배열
  monthly_goal smallint,          -- 월 독서 목표 권수
  notify_enabled boolean default true,
  created_at  timestamptz default now()
);
```

---

## 네비게이션

```
회원가입 성공 → Onboarding (백스택 제거)
회원가입 실패 → 에러 Snackbar 표시
로그인 화면으로 이동 → Login (텍스트 링크)
```

---

## TODO

### UI 구현
- [ ] `SignUpScreen` Composable 구현 (Light / Dark)
- [ ] 이메일 + 비밀번호 + 닉네임 입력 폼
- [ ] Google / Apple 소셜 가입 버튼
- [ ] `SignUpViewModel` / MVI State, Action, SideEffect 정의
- [ ] 가입 성공 → Onboarding 네비게이션
- [ ] 이메일 중복 / 유효성 검사 에러 처리

### 온보딩 플로우 구현
- [ ] `OnboardingScreen` (Step 1~3) Composable 구현
- [ ] 독서 목표 선택 UI (슬라이더 or 버튼)
- [ ] 선호 장르 멀티 선택 칩 UI
- [ ] 출생연도 드롭다운 / 직업군 선택 UI
- [ ] "건너뛰기" 버튼 처리
- [ ] 온보딩 완료 데이터 → `profiles` 테이블 upsert

### Supabase 연동
- [ ] `profiles` 테이블 생성 (위 스키마 참고)
- [ ] Row Level Security (RLS) 정책: 본인 row만 read/write 허용
- [ ] 이메일/비밀번호 회원가입 연동
- [ ] 소셜 가입 후 `profiles` 자동 생성 (DB Function or Edge Function 활용)
