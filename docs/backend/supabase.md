# Supabase

## 구성 요소

| 구성 요소 | 사용 목적 |
|---|---|
| **PostgreSQL** | 도서 정보, 독서 기록, 사용자 데이터 저장 |
| **Auth** | 이메일 / Google / Apple 소셜 로그인 |
| **Storage** | 도서 표지 이미지 업로드 및 저장 |
| **Realtime** | 독서 진행 상태 실시간 동기화 (선택) |
| **Edge Functions** | FCM 발송 등 서버사이드 로직 처리 |

---

## 환경 분리 (dev / prod)

빌드 타입에 따라 Supabase 프로젝트를 분리한다.

| 빌드 타입 | Supabase 프로젝트 | applicationId |
|---|---|---|
| `debug` | dev 프로젝트 | `com.winstonmoon.legato.dev` |
| `release` | prod 프로젝트 | `com.winstonmoon.legato` |

**시크릿 주입 방식**

- **Android**: `buildTypes`에 `buildConfigField`로 주입 → `BuildConfig.SUPABASE_URL` / `SUPABASE_ANON_KEY`
- **iOS / commonMain**: `generateSupabaseConfig` Gradle 태스크가 빌드 시 `SupabaseConfig.kt`를 생성하여 commonMain에 주입 (기존 `generateApiConfig` 패턴과 동일)
- **시크릿 관리**: `local.properties`에 4개 값 저장 → VCS 미커밋

```properties
# local.properties
supabase.dev.url=https://YOUR_DEV_PROJECT.supabase.co
supabase.dev.anon_key=YOUR_DEV_ANON_KEY
supabase.prod.url=https://YOUR_PROD_PROJECT.supabase.co
supabase.prod.anon_key=YOUR_PROD_ANON_KEY
```

---

## SDK 설정

```kotlin
// build.gradle.kts (commonMain)
implementation("io.github.jan-tennert.supabase:postgrest-kt:$supabaseVersion")
implementation("io.github.jan-tennert.supabase:auth-kt:$supabaseVersion")
implementation("io.github.jan-tennert.supabase:storage-kt:$supabaseVersion")
implementation("io.github.jan-tennert.supabase:realtime-kt:$supabaseVersion")

// Ktor Engine (platformModule)
// androidMain
implementation("io.ktor:ktor-client-okhttp:$ktorVersion")
// iosMain
implementation("io.ktor:ktor-client-darwin:$ktorVersion")
```

```kotlin
// SupabaseConfig.kt — generateSupabaseConfig 태스크로 빌드 시 자동 생성됨
// Android는 BuildConfig.SUPABASE_URL / BuildConfig.SUPABASE_ANON_KEY 사용 가능
// iOS / commonMain은 SupabaseConfig 객체를 직접 참조
val supabaseClient = createSupabaseClient(
    supabaseUrl = SupabaseConfig.DEV_URL,   // 실제로는 빌드 환경에 맞는 값 선택
    supabaseKey = SupabaseConfig.DEV_ANON_KEY,
) {
    install(Auth)
    install(Postgrest)
    install(Storage)
}
```

---

## 데이터베이스 스키마

### users

```sql
create table users (
  id uuid references auth.users primary key,
  email text,
  display_name text,
  avatar_url text,
  created_at timestamptz default now()
);
```

### books

```sql
create table books (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references users(id) on delete cascade,
  title text not null,
  author text not null,
  publisher text,
  total_pages int,
  cover_url text,
  isbn text,
  status text not null default 'WISHLIST',
  current_page int default 0,
  started_at timestamptz,
  finished_at timestamptz,
  created_at timestamptz default now()
);

-- RLS 정책
alter table books enable row level security;
create policy "Users can manage own books"
  on books for all
  using (auth.uid() = user_id);
```

### reading_sessions

```sql
create table reading_sessions (
  id uuid primary key default gen_random_uuid(),
  book_id uuid references books(id) on delete cascade,
  start_page int,
  end_page int,
  duration_min int,
  session_date date not null,
  memo text,
  created_at timestamptz default now()
);

alter table reading_sessions enable row level security;
create policy "Users can manage own sessions"
  on reading_sessions for all
  using (
    auth.uid() = (select user_id from books where id = book_id)
  );
```

---

## 인증 흐름

```
앱 실행
  → supabaseClient.auth.currentSessionOrNull()
  → 세션 있음: Home으로 이동
  → 세션 없음: Login 화면

로그인
  → Google OAuth: supabaseClient.auth.signInWith(Google)
  → Apple OAuth:  supabaseClient.auth.signInWith(Apple)
  → 이메일:       supabaseClient.auth.signInWith(Email) { ... }
  → JWT 발급 → 세션 저장 → Home으로 이동
```

---

## Storage — 도서 표지 이미지

```kotlin
// 이미지 업로드
supabaseClient.storage["book-covers"].upload(
    path = "user_id/book_id.jpg",
    data = imageBytes,
    upsert = true,
)

// 공개 URL 획득
val url = supabaseClient.storage["book-covers"]
    .publicUrl("user_id/book_id.jpg")
```

---

## Edge Function — FCM 발송

```javascript
// supabase/functions/send-notification/index.ts
Deno.serve(async (req) => {
  const { userId, title, body } = await req.json()
  // FCM HTTP v1 API 호출
  await fetch("https://fcm.googleapis.com/v1/...", { ... })
  return new Response("ok")
})
```

!!! note "FCM 서버 불필요"
    Supabase Edge Function으로 FCM 발송을 처리하므로 별도 서버가 필요 없다.

---

## TODO

### 프로젝트 초기 설정
- [ ] Supabase Dashboard에서 dev / prod 프로젝트 각각 생성
- [ ] `local.properties`에 4개 값 입력 (`supabase.dev.url`, `supabase.dev.anon_key`, `supabase.prod.url`, `supabase.prod.anon_key`)
- [ ] `supabase-kt` 의존성 추가 — `postgrest-kt`, `auth-kt`, `storage-kt`, `realtime-kt`
- [ ] `SupabaseClient` 싱글턴 구현 — 빌드 타입에 맞는 URL / anon key 선택
- [ ] Koin DI에 `SupabaseClient` 싱글턴 등록

### 데이터베이스 스키마
- [ ] `users` 테이블 생성 + RLS 정책 설정
- [ ] `books` 테이블 생성 + RLS 정책 설정
- [ ] `reading_sessions` 테이블 생성 + RLS 정책 설정
- [ ] `profiles` 테이블 생성 (닉네임, 출생연도, 직업군, 장르, 월간 목표) + RLS 정책
- [ ] `genres` 테이블 생성 (SearchScreen 장르 칩 원격 fetch용, 다국어 필드 포함)
- [ ] 초기 장르 18개 데이터 삽입

### 인증 (Auth)
- [ ] Supabase Dashboard에서 이메일/비밀번호 Auth provider 활성화
- [ ] `LoginViewModel` — 이메일 로그인 `supabaseClient.auth.signInWith(Email)` 연동
- [ ] `SignupViewModel` — 이메일 회원가입 `supabaseClient.auth.signUpWith(Email)` 연동
- [ ] `OnboardingViewModel` — 가입 후 `profiles` 테이블 upsert 연동
- [ ] 앱 시작 시 세션 자동 복원 (`currentSessionOrNull()` → Home/Login 분기)
- [ ] Google OAuth 연동 (Android `google-services.json`, iOS `GoogleService-Info.plist`)
- [ ] Apple OAuth 연동 (iOS 전용 expect/actual, Xcode Entitlement 추가)
- [ ] Anonymous Auth (게스트 로그인) 활성화 및 `signInAnonymously()` 연동
- [ ] `ProfileScreen` — 로그아웃 `supabaseClient.auth.signOut()` 연동

### 데이터 동기화 (Remote Repository)
- [ ] `RemoteBookRepository` 구현 (Supabase postgrest-kt로 `books` 테이블 CRUD)
- [ ] `RemoteReadingSessionRepository` 구현 (`reading_sessions` 테이블 CRUD)
- [ ] Local → Remote 마이그레이션 전략 결정 (로컬 Room DB와 Supabase 동시 운영 or 전환)
- [ ] 오프라인 우선 전략: Room DB를 캐시로 유지 + Supabase 동기화

### Storage
- [ ] Supabase Dashboard에서 `book-covers` 버킷 생성 (public 또는 signed URL 방식 결정)
- [ ] `ManualEntryScreen` 표지 이미지 업로드 → `book-covers/{userId}/{bookId}.jpg`
- [ ] 업로드 후 공개 URL을 `books.cover_url`에 저장

### Edge Functions
- [ ] `send-notification` Edge Function 작성 및 배포 (FCM HTTP v1 API 호출)
- [ ] Supabase Dashboard에서 FCM Service Account JSON을 Vault에 등록
- [ ] 앱에서 Edge Function 호출 로직 구현 (푸시 알림 트리거)
