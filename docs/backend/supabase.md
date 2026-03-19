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
// Supabase 클라이언트 초기화
val supabaseClient = createSupabaseClient(
    supabaseUrl = BuildConfig.SUPABASE_URL,
    supabaseKey = BuildConfig.SUPABASE_ANON_KEY,
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
