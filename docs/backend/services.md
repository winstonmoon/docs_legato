# Firebase & AdMob

## Firebase 설정

KMP에서 Firebase는 [GitLiveApp/firebase-kotlin-sdk](https://github.com/GitLiveApp/firebase-kotlin-sdk)를 사용한다.

```kotlin
// build.gradle.kts (commonMain)
implementation("dev.gitlive:firebase-crashlytics:$firebaseKotlinVersion")
implementation("dev.gitlive:firebase-analytics:$firebaseKotlinVersion")
implementation("dev.gitlive:firebase-messaging:$firebaseKotlinVersion")
```

!!! warning "iOS dSYM 업로드 필요"
    Kotlin/Native 스택 트레이스 정상 출력을 위해 Xcode 빌드 페이즈에서 dSYM 파일을 Firebase에 업로드해야 한다.

---

## Firebase Crashlytics

앱 크래시 및 비치명적 오류 자동 수집.

### 설정

```kotlin
val crashlytics = Firebase.crashlytics

// 사용자 ID 연동 (Supabase Auth userId)
crashlytics.setUserId(userId)
```

### 수동 로깅

```kotlin
// Non-fatal 예외 기록
try {
    booksApiService.searchBooks(query)
} catch (e: Exception) {
    Firebase.crashlytics.recordException(e)
    // 에러 UI 표시
}

// Breadcrumb 로그
Firebase.crashlytics.log("User clicked Add to Library")
```

### 적용 포인트

| 위치 | 로깅 방법 |
|---|---|
| Google Books API 오류 | `recordException(e)` |
| Supabase 오류 | `recordException(e)` |
| 파일 저장 실패 | `log("...") + recordException(e)` |

---

## Firebase Analytics

사용자 행동 데이터 수집.

### 주요 커스텀 이벤트

| 이벤트 | 설명 | 파라미터 |
|---|---|---|
| `book_added` | 도서 추가 | `source`: `"search"` / `"manual"` |
| `reading_session_saved` | 독서 세션 저장 | `pages_read`, `book_id` |
| `book_completed` | 도서 완독 | `book_id`, `total_pages` |
| `search_performed` | 검색 실행 | `query`, `result_count` |
| `screen_view` | 화면 진입 | `screen_name` (자동 수집) |

### 구현

```kotlin
val analytics = Firebase.analytics

// 도서 추가 이벤트
analytics.logEvent("book_added") {
    param("source", "search")
}

// 완독 이벤트
analytics.logEvent("book_completed") {
    param("book_id", bookId)
    param("total_pages", totalPages.toLong())
}
```

---

## Firebase Cloud Messaging (FCM)

독서 알림 (목표 페이지, 독서 스트릭 등) 발송.

### 대안 라이브러리: KMPNotifier

```kotlin
// build.gradle.kts (commonMain)
implementation("io.github.mirzemehdi:kmpnotifier:$version")
```

기능:
- 토큰 발급 및 갱신
- 토픽 구독 / 해제
- 로컬 알림 지원

### FCM 토큰 등록

```kotlin
// 로그인 후 토큰을 Supabase에 저장
val token = NotifierManager.getPushNotifier().getToken()
supabaseClient.from("users").update(mapOf("fcm_token" to token)) { ... }
```

### 서버 발송

Supabase Edge Function에서 FCM HTTP v1 API를 호출 (별도 서버 불필요).

---

## Google AdMob

### 광고 유형 및 위치

| 유형 | 위치 | 조건 |
|---|---|---|
| 배너 (Banner) | Library / Search 화면 하단 | 비프리미엄 사용자 |
| 전면 (Interstitial) | 도서 추가 완료 후 | 3회마다 1번 노출 |
| 보상형 (Rewarded) | 추후 검토 | — |

### KMP AdMob 구현 방법

!!! warning "공식 KMP 지원 없음"
    AdMob 공식 SDK는 KMP를 지원하지 않으므로 `expect/actual`로 플랫폼별 직접 구현이 필요하다.

```kotlin
// commonMain
expect class AdManager {
    fun showBanner(placementId: String)
    fun showInterstitial(onDismissed: () -> Unit)
}

// androidMain
actual class AdManager {
    actual fun showBanner(placementId: String) {
        // Google Mobile Ads SDK
    }
    actual fun showInterstitial(onDismissed: () -> Unit) { ... }
}

// iosMain
actual class AdManager {
    actual fun showBanner(placementId: String) {
        // GADBannerView
    }
    actual fun showInterstitial(onDismissed: () -> Unit) { ... }
}
```

### GDPR 동의 (UMP SDK)

```kotlin
// Android
val consentRequestParameters = ConsentRequestParameters.Builder()
    .setTagForUnderAgeOfConsent(false)
    .build()
ConsentInformation.getInstance(context).requestConsentInfoUpdate(...)
```

!!! warning "GDPR 필수"
    EU 사용자 대상으로 AdMob을 사용할 경우 UMP(User Messaging Platform) SDK 연동이 필수다.
