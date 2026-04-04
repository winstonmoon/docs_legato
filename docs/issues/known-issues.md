# 알려진 이슈 & 기술 부채

> 2026-03-13 코드 리뷰에서 발견된 문제점 목록

---

## 🔴 HIGH — 즉시 해결 필요

### 1. API 키 바이너리 노출 ✅ 수정 완료

- **문제:** `const val GOOGLE_BOOKS_API_KEY`가 `ApiConfig.kt`에 박히고, 컴파일 시 인라이닝 → APK/IPA 디컴파일하면 키 추출 가능
- **원인:** `local.properties`로 VCS는 막지만 빌드 결과물에는 그대로 남음
- **해결:** `const val` → `val` 변경 + ProGuard/R8 minification 적용 (2026-03-13 수정 완료)

```kotlin
// ❌ 잘못된 방법
const val GOOGLE_BOOKS_API_KEY = BuildConfig.API_KEY // 인라이닝됨

// ✅ 올바른 방법
val GOOGLE_BOOKS_API_KEY = BuildConfig.API_KEY // 인라이닝 방지
```

---

## 🟡 MEDIUM — 다음 스프린트 내 해결

### 2. 설정 영속성 — 부분 완료 ⚠️

- **테마 / 언어:** `DataStoreAppPreferences`로 DataStore 영속화 완료. 앱 재시작 후에도 설정 유지됨.
- **로그인 상태 (`isLoggedIn`):** DataStore에 저장하지 않아 앱 재시작 시 초기화됨. Supabase 세션 자동 복원 연동 전까지는 매번 로그인 필요.
- **해결 필요:** `isLoggedIn` 영속화 또는 앱 시작 시 Supabase `currentSessionOrNull()` 세션 복원 로직 연동

---

### 3. HTTP Client 리소스 누수

- **문제:** Ktor `HttpClient`가 Koin scope 종료 시 닫히지 않음
- **해결:** Koin 모듈에 `onClose { client.close() }` 추가 필요

```kotlin
// ❌ 문제 있는 코드
single<HttpClient> { HttpClient(OkHttp) { ... } }

// ✅ 해결 방법
single<HttpClient> {
    HttpClient(OkHttp) { ... }
} onClose {
    it?.close()
}
```

---

### 4. Android / iOS 네비게이션 불일치

- **문제:** Android는 `androidx.navigation.compose`, iOS는 커스텀 `StackAppNavigator`
- **영향:** `BookDetail` / `BookEdit`이 iOS에서 `PlaceholderDetail`로만 표시됨
- **해결:** 공통 인터페이스 기반으로 통일하거나 iOS 구현 완성 필요

---

### 5. 페이지네이션 미구현

- **문제:** `page` 파라미터가 있으나 항상 `page=0`만 사용, 검색 결과 20개 한계
- **해결:** `LoadState` + 무한 스크롤 또는 페이지 버튼 구현 필요

```kotlin
// 구현 필요
fun observeBooksPage(
    query: String,
    page: Int,
    pageSize: Int = 20,
): Flow<List<SearchResult>>
```

---

## 🔵 LOW — 백로그

### 6. 에러 메시지 UI 직접 노출

- **문제:** `exception.message` 그대로 사용자에게 표시됨
- **해결:** 에러 타입별 사용자 친화적 메시지로 변환 필요

```kotlin
// ❌ 나쁜 예
Text(error.message ?: "Unknown error")

// ✅ 좋은 예
Text(error.toUserFriendlyMessage())
```

---

### 7. 미사용 코드 ✅ 수정 완료

- `Greeting.kt`, `Platform.kt` 파일 삭제 완료 (2026-04-05)
- 빈 PlatformModule 파일들 정리 완료

---

### 8. 테스트 없음

| 대상 | 우선순위 |
|---|---|
| ViewModel (Unit Test) | 높음 |
| Repository (Integration Test) | 높음 |
| API 서비스 (Unit Test) | 중간 |
| UI (Screenshot Test) | 낮음 |

```kotlin
// ViewModel 테스트 예시 구조
class HomeViewModelTest {
    @Test
    fun `현재 읽는 책이 없으면 빈 상태를 표시한다`() {
        // given
        val repository = FakeBookRepository(emptyList())
        val viewModel = HomeViewModel(repository)

        // when & then
        viewModel.uiState.test {
            assertThat(awaitItem().currentBook).isNull()
        }
    }
}
```

---

### 9. Release 빌드 난독화 없음

- **문제:** `isMinifyEnabled = false` (2026-03-13 HIGH 수정 시 함께 해결 완료)

---

## 체크리스트

| 이슈 | 우선순위 | 상태 |
|---|---|---|
| API 키 바이너리 노출 | 🔴 HIGH | ✅ 완료 |
| Release 난독화 없음 | 🔴 HIGH | ✅ 완료 |
| 설정 영속성 (테마/언어 완료, isLoggedIn 미완) | 🟡 MEDIUM | ⚠️ 부분 완료 |
| HTTP Client 리소스 누수 | 🟡 MEDIUM | ⬜ 미완 |
| 네비게이션 불일치 | 🟡 MEDIUM | ⬜ 미완 |
| 페이지네이션 미구현 | 🟡 MEDIUM | ⬜ 미완 |
| 에러 메시지 직접 노출 | 🔵 LOW | ⬜ 미완 |
| 미사용 코드 | 🔵 LOW | ✅ 완료 |
| 테스트 없음 | 🔵 LOW | ⬜ 미완 |
