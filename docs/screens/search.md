# 검색 & 도서 추가 (Search & Add Book)

[:fontawesome-solid-arrow-up-right-from-square: Stitch에서 보기](https://stitch.withgoogle.com/projects/9973353658769610659){ target="_blank" .md-button }

---

## 화면 상태 개요

| 상태 | 설명 |
|---|---|
| `Initial` | 검색어 없음 — 최근 검색어 · 장르 탐색 · 이번 주 인기 도서 표시 |
| `Loading` | Google Books API 요청 중 |
| `Results` | 검색 결과 목록 (필터 바 포함) |
| `NoResults` | 결과 없음 → "수동으로 입력하기" 유도 |
| `Error` | 네트워크 오류 안내 메시지 |

---

## Initial State UI 구성

> Stitch 디자인: **Search - Initial (Light)** / **Search - Initial with Barcode Icon (Dark)**

| 구성 요소 | 설명 |
|---|---|
| 검색 바 | 탭 시 검색 입력 모드로 전환; 오른쪽 끝에 **바코드 스캐너 아이콘** (`barcode_scanner`) 배치 |
| Hero 섹션 | 마스코트 일러스트 + "Find your next story" 헤딩 + 서브 텍스트 |
| Recent Searches | 최근 검색어 도서 카드 목록 (표지, 제목, 저자, `history` 아이콘) + "Clear all" 버튼 |
| Explore Genres | 장르 필터 칩 가로 스크롤 목록 |
| Popular This Week | 이번 주 인기 도서 카드 (`orderBy=newest` API 기반, 표지 + 제목 + 저자) |
| SearchCtaSection | 직접 입력 유도 배너 — "Can't find what you're looking for?" + "Add it manually" 버튼 |
| 하단 내비게이션 | Home · Search · Library · Profile |

!!! note "바코드 스캐너 아이콘"
    검색 바 오른쪽 `barcode_scanner` 아이콘 탭 → **ISBN 바코드 스캔** 기능 실행.
    이미 구현된 `IsbnBarcodeScanner` expect/actual과 연결.

### Stitch 스크린샷

=== "Light"
    ![Search Initial Light](https://lh3.googleusercontent.com/aida/ADBb0ujX2VTOwnr_JHXKwyjIZ6x-rnZ7t-xJh6CQv06gxttR_bu62s7DIRx9-9HTiF_rjaMuJdBvF5SY-t6N3BQHWu6ExL7T23E3EMZLKCQS2qCQpG08vyCWXzuMEtl-3XoU6HnJDow4xVfHMeCsIqfC1pdwjoQJD1dfcWHFxjBYKSpT57EUdPrsfU_md86oSKlElYrRchgoLU3ffkuksbfsK7SAZbGEYw97aQtjCUnHvNwe0e491rzjMnAWZyM)

=== "Dark"
    ![Search Initial Dark](https://lh3.googleusercontent.com/aida/ADBb0uj7tvHGE_VDi0_e-brpBGh7D53gkymoxNmgEz9m3tMLAdR1P-ydi5x68As3FLXUhnlCRZLTc7LhKCPGPWpv4DzPX7pUDMa51am8UEda9xPwTSeov48pBjbp96VhMoMUfFji34n0IP03kz_58xG9BadmVrlxPleS0EEmzA2AesMbQ0EaUivijGq_oBvDLEEkNoSrSoDzOCAO3qWMAVgjtovy1LuJC2UOs6hD6yYga5V7HlXIGHszw39qdQ)

---

## Search Results UI 구성

> Stitch 디자인: **Search and Add Book Updated** / **Search and Add (Light)**

| 구성 요소 | 설명 |
|---|---|
| 검색 입력 필드 | 뒤로가기 버튼 + 검색 입력창 |
| 정렬 칩 | **관련도** · **최신순** (Google Books API 지원 2종) |
| 검색 결과 리스트 | 책 표지, 제목, 저자, 별점(★), 출판 연도, Add/Added 버튼 |
| ManualEntryFooter | 검색 결과 목록 최하단 — "찾는 책이 없으신가요?" + 직접 입력 유도 CTA |
| 하단 내비게이션 | Home · Search · Library · Profile |

!!! note "정렬 옵션 — Google Books API 제약"
    Google Books API의 `orderBy` 파라미터는 `relevance`(기본값)와 `newest` **2종만** 공식 지원합니다.
    `제목순` 정렬은 API 레벨에서 지원되지 않으므로 UI에서 제거되었습니다.

    | 칩 | `orderBy` 값 | API 지원 |
    |---|---|---|
    | 관련도 | _(파라미터 생략)_ | ✅ |
    | 최신순 | `newest` | ✅ |

### Stitch 스크린샷

=== "Dark"
    ![Search and Add Updated Dark](https://lh3.googleusercontent.com/aida/ADBb0uiRZhVVCil0kzOTNdp5BEuKXdds1PlQFJ5nYQfwaSblu2MtxkL05_N0JjJUF_yW7MoPEVnvYHMYQX4Puv3qrUF61dwqJrA0NGdhwiznj22WMKoSPaZXQW5t6FVjtDDCpL8DzXBC-OVTjdo0cpjWGw2IMKIt9DPp8ut5YwAMcE-fWaORLcBzNONHkGfsPKc408JX6rULwQFmxsuuryouSvX9H8XJILCBVq2USDzFcirit4YJheB9oNprTYk)

=== "Light"
    ![Search and Add Light](https://lh3.googleusercontent.com/aida/ADBb0ugISkI2gvNnOIMqhf0G8T9jINGYr9j-9P1OnIRzoFN8I2KGEh108f59tRbVXLrNFAcj2S7xD9DNePx316U8YQZl4BdGqiRMIIfGxB6vdtuSiJ0GOSqr6xvPiOsGX1RLr1jKmhKVrTnjpTMcOYsaTIA5dk9sXaUaqa4KIEZviOxrOYWL5Lq3dMMed1esBb-M5xVjzEO0PBUObwtm2WhwDbi5qBFbQasdhXO8Y5YoUMd2uOTZUHQy8NryckA)

---

## No Results 상태 UI 구성

> Stitch 디자인: **Search No Results (Light) - with Manual Entry button** / **Search No Results (Dark) - with Manual Entry button**

| 구성 요소 | 설명 |
|---|---|
| 마스코트 일러스트 | 물음표가 붙은 당황한 책 캐릭터 |
| "No Results Found" 헤딩 | 검색 결과 없음 안내 |
| 안내 텍스트 | "We couldn't find any books matching your search." |
| **직접 입력하기** 버튼 | 탭 시 Manual Book Entry 화면으로 이동 |
| "Clear search history" 링크 | 검색 기록 전체 삭제 |
| 하단 내비게이션 | Home · Search · Library · Profile |

!!! note "화면 이동 플로우"
    `NoResults` 상태에서 **직접 입력하기** 버튼 탭 → `SearchSideEffect.NavigateToManualEntry` 발행
    → `ManualBookEntryScreen`으로 이동.

### Stitch 스크린샷

=== "Light"
    ![Search No Results Light](https://lh3.googleusercontent.com/aida/ADBb0uj8gb-FHA6VNltsDw6owk2zXRvu0naHk9hWmJlOB4PZnnjAD-JXBPR4bXc9jypyStAwVZV__wzgOLvMlf1p-evJgjYO4qpBgElsm2zkULI4RjoPy089iDz4t9gCNS21LMzsxOAkDEliS_rFIRG2ZrArCRa6oNVPRDJzkB-KycdaG014r-FdTPeAV4kq8nO7wWqz4hFBqGuRSnnycKAAmxPs0i5a5iBvYnKnAFVDnbkpG_ZNA6Ao5L4AVg)

=== "Dark"
    ![Search No Results Dark](https://lh3.googleusercontent.com/aida/ADBb0ujcQnywq2oNdSe7Q5tWnDeOmwxCH_aAtyIgzA9E6PBkEz9sc0QoKDUfhzrWBkHeXtV-ll1VI8TBv4fOKyRavW69gpWibQNE0RCJ3ZfUeRgiWS-X8ZfVIPQrVoBilh6sS0nYA7ErG3xSZXkjXhHVfVSE5rf241D_IWHOjhgb9HStUxiVnXSEcGygN_u0_iM2tLIZ2CbpdxAZU2M0PZlzrG_sBzQtlDN6VMzA7ty8wQ7_bNERRRG2uR-YnI8)

---

## Explore Genres — 장르 칩

### 장르 목록 관리 방식: Supabase 원격 fetch

!!! info "Google Books API에는 전용 장르 목록 엔드포인트가 없습니다"
    `/genres` 또는 `/categories` 전용 API는 제공되지 않습니다.
    대신 **`subject:` 필터**를 이용해 특정 장르의 책을 검색합니다.

    ```
    GET https://www.googleapis.com/books/v1/volumes
      ?q=subject:Fiction
      &orderBy=relevance
      &maxResults=20
    ```

장르 목록 자체는 **Supabase `genres` 테이블**에서 원격으로 fetch한다.
DB에서 장르를 추가·수정·비활성화(`is_active=false`)하면 앱 재배포 없이 즉시 반영된다.

**fetch 전략 (3단계 fallback):**

1. **원격 fetch** — Supabase PostgREST API (`GET /rest/v1/genres?is_active=eq.true&order=sort_order`)
2. **로컬 캐시** — fetch 성공 시 DataStore에 JSON 저장, 실패 시 캐시 사용
3. **하드코딩 fallback** — 캐시도 없을 경우 앱 내장 기본 목록 사용

### Supabase `genres` 테이블 스키마

| 컬럼 | 타입 | 설명 |
|---|---|---|
| `id` | int | PK |
| `key` | text | Google Books `subject:` 쿼리 값 (예: `Fiction`) |
| `emoji` | text | UI 칩 이모지 |
| `label_en` | text | 영어 레이블 |
| `label_ko` | text | 한국어 레이블 |
| `label_ja` | text | 일본어 레이블 |
| `label_zh` | text | 중국어 레이블 |
| `is_active` | bool | false 시 앱에서 미표시 |
| `sort_order` | int | 칩 정렬 순서 |

### 초기 장르 목록 18개 (BISAC 기반)

| EN | 한국어 | 日本語 | 中文 | Google Books `subject` 쿼리 |
|---|---|---|---|---|
| Fiction | 소설 | フィクション | 小说 | `subject:Fiction` |
| Mystery | 미스터리 | ミステリー | 悬疑 | `subject:Mystery` |
| Sci-Fi | SF | SF | 科幻 | `subject:Science Fiction` |
| Fantasy | 판타지 | ファンタジー | 奇幻 | `subject:Fantasy` |
| Romance | 로맨스 | ロマンス | 言情 | `subject:Romance` |
| Thriller | 스릴러 | スリラー | 惊悚 | `subject:Thriller` |
| Horror | 공포 | ホラー | 恐怖 | `subject:Horror` |
| Biography | 전기 | 伝記 | 传记 | `subject:Biography & Autobiography` |
| History | 역사 | 歴史 | 历史 | `subject:History` |
| Self-Help | 자기계발 | 自己啓発 | 励志 | `subject:Self-Help` |
| Business | 경제경영 | ビジネス | 商业 | `subject:Business & Economics` |
| Psychology | 심리학 | 心理学 | 心理学 | `subject:Psychology` |
| Science | 과학 | 科学 | 科学 | `subject:Science` |
| Philosophy | 철학 | 哲学 | 哲学 | `subject:Philosophy` |
| True Crime | 실화 범죄 | 実話犯罪 | 真实犯罪 | `subject:True Crime` |
| Comics | 만화 | コミック | 漫画 | `subject:Comics & Graphic Novels` |
| Young Adult | 청소년 | ヤングアダルト | 青少年 | `subject:Young Adult Fiction` |
| Children's | 어린이 | 児童 | 儿童 | `subject:Juvenile Fiction` |

!!! note "API key 수정 사항"
    기존 하드코딩 대비 3개 key 수정 (BISAC 정확 일치로 검색 범위 확대):

    | 기존 key | 변경 key |
    |---|---|
    | `Biography` | `Biography & Autobiography` |
    | `Business` | `Business & Economics` |
    | `Children` | `Juvenile Fiction` |

### 아키텍처

```kotlin
// Genre 도메인 모델 (다국어 필드 포함)
data class Genre(
    val key: String,
    val emoji: String,
    val labelEn: String,
    val labelKo: String,
    val labelJa: String,
    val labelZh: String,
    val sortOrder: Int,
)

// 현재 언어 설정에 따라 displayName 자동 선택
fun Genre.displayName(language: AppLanguage): String = when (language) {
    AppLanguage.KOREAN  -> labelKo
    AppLanguage.JAPANESE -> labelJa
    AppLanguage.CHINESE -> labelZh
    else                -> labelEn
}
```

```kotlin
// GenreRepository — fetch → 캐시 → fallback
class GenreRepository(
    private val apiService: GenreApiService,
    private val dataStore: DataStore<Preferences>,
) {
    suspend fun fetchGenres(): List<Genre>  // 원격 fetch, 캐시 저장
    fun getCachedGenres(): List<Genre>      // DataStore에서 읽기
    fun getFallbackGenres(): List<Genre>    // 하드코딩 기본값
}
```

---

## Recent Searches — 최근 검색어

### 구현 방식: DataStore (Preferences)

Room DB까지는 불필요하며, 소규모 문자열 리스트를 저장하기에 **DataStore + kotlinx.serialization JSON**이 적합합니다.

```kotlin
@Serializable
data class RecentSearch(
    val query: String,
    val timestamp: Long = System.currentTimeMillis()
)

@Serializable
data class RecentSearchList(
    val searches: List<RecentSearch> = emptyList()
)
```

```kotlin
// RecentSearchRepository
class RecentSearchRepository(private val dataStore: DataStore<Preferences>) {

    private val KEY = stringPreferencesKey("recent_searches")
    private val MAX_COUNT = 10

    val recentSearchesFlow: Flow<List<RecentSearch>> = dataStore.data
        .map { prefs ->
            prefs[KEY]
                ?.let { Json.decodeFromString<RecentSearchList>(it).searches }
                ?: emptyList()
        }

    suspend fun addSearch(query: String) {
        val trimmed = query.trim()
        if (trimmed.isBlank()) return
        dataStore.edit { prefs ->
            val current = prefs[KEY]
                ?.let { Json.decodeFromString<RecentSearchList>(it).searches }
                ?.toMutableList() ?: mutableListOf()
            // 중복 제거 후 맨 앞에 추가, 최대 10개 유지
            current.removeAll { it.query == trimmed }
            current.add(0, RecentSearch(trimmed))
            prefs[KEY] = Json.encodeToString(RecentSearchList(current.take(MAX_COUNT)))
        }
    }

    suspend fun removeSearch(query: String) {
        dataStore.edit { prefs ->
            val current = prefs[KEY]
                ?.let { Json.decodeFromString<RecentSearchList>(it).searches }
                ?.filter { it.query != query } ?: emptyList()
            prefs[KEY] = Json.encodeToString(RecentSearchList(current))
        }
    }

    suspend fun clearAll() {
        dataStore.edit { prefs -> prefs.remove(KEY) }
    }
}
```

### DI 등록 (Koin)

```kotlin
// di/DataModule.kt
val dataModule = module {
    single {
        PreferenceDataStoreFactory.create(
            corruptionHandler = ReplaceFileCorruptionHandler { emptyPreferences() },
            produceFile = { context.dataStoreFile("recent_searches.preferences_pb") }
        )
    }
    single { RecentSearchRepository(get()) }
}
```

---

## MVI 구조

```kotlin
data class SearchUiState(
    val query: String = "",
    val searchState: SearchState = SearchState.Initial,
    val recentSearches: List<RecentSearch> = emptyList(),
    val selectedGenre: BookGenre? = null,
    val filters: SearchFilters = SearchFilters(),
)

data class SearchFilters(
    val genre: BookGenre? = null,
    val author: String? = null,
    val minRating: Float? = null,
    val year: Int? = null,
)

sealed interface SearchState {
    data object Initial : SearchState
    data object Loading : SearchState
    data class Results(val books: List<SearchResultUiModel>) : SearchState
    data object NoResults : SearchState
    data class Error(val message: String) : SearchState
}

sealed interface SearchAction {
    data class QueryChange(val query: String) : SearchAction
    data class GenreClick(val genre: BookGenre) : SearchAction
    data class RecentSearchClick(val query: String) : SearchAction
    data class RemoveRecentSearch(val query: String) : SearchAction
    data object ClearRecentSearches : SearchAction
    data class FilterChange(val filters: SearchFilters) : SearchAction
    data class AddToLibraryClick(val book: SearchResultUiModel) : SearchAction
    data object ManualEntryClick : SearchAction
}

sealed interface SearchSideEffect {
    data object NavigateToManualEntry : SearchSideEffect
    data object ShowAddSuccessToast : SearchSideEffect
    data object ShowInterstitialAd : SearchSideEffect
}
```

---

## Google Books API 연동

```kotlin
// 텍스트 검색 (Debounce 300ms)
queryFlow
    .debounce(300)
    .filter { it.length >= 2 }
    .flatMapLatest { query -> bookApiService.searchBooks(query) }

// 장르 탐색
suspend fun searchByGenre(genre: BookGenre): List<Book> {
    return bookApiService.searchBooks(
        query = genre.subjectQuery,
        orderBy = "relevance"
    )
}

// API 엔드포인트
GET https://www.googleapis.com/books/v1/volumes
  ?q={query}          // 텍스트 검색 또는 "subject:Fiction"
  &orderBy=relevance  // relevance | newest
  &key={API_KEY}
  &maxResults=20
```

!!! warning "API 키 보안"
    `local.properties`에 저장하고 `BuildConfig`를 통해 주입.
    `const val` 대신 `val`을 사용하여 APK 인라이닝 방지.

---

## AdMob — 도서 추가 완료 후 Interstitial

```kotlin
// 도서 추가 성공 시
addCount++
if (addCount % 3 == 0) { // 3회마다 1번
    showInterstitialAd()
}
```

---

## Analytics 이벤트

| 이벤트 | 파라미터 |
|---|---|
| `search_performed` | `query`, `result_count` |
| `genre_explored` | `genre` |
| `recent_search_used` | `query` |
| `book_added` | `source = "search" \| "genre"` |

---

## TODO

- [x] `SearchScreen` — Initial State Composable 구현 (Light / Dark)
    - [x] 검색 바 오른쪽에 `barcode_scanner` 아이콘 배치 → ISBN 바코드 스캔 연동
    - [ ] Hero 섹션 (마스코트 이미지 + "Find your next story" 텍스트)
    - [x] Recent Searches 섹션 (도서 카드 + `history` 아이콘 + "Clear all")
    - [x] Explore Genres 섹션 (가로 스크롤 칩 목록, `BookGenre` enum 기반)
    - [x] Popular This Week 섹션 (`orderBy=newest` 기반 인기 도서 카드)
    - [x] SearchCtaSection — 직접 입력 유도 배너 ("Add it manually" 버튼)
- [x] `SearchScreen` — Results State Composable 구현
    - [x] 정렬 칩: 관련도 · 최신순 (Google Books API 지원 2종, `SortOrder` enum 기반)
    - [x] 검색 결과 아이템 (표지, 제목, 저자, 별점, 출판 연도, Add/Added 버튼)
    - [x] ManualEntryFooter — 검색 결과 최하단 직접 입력 유도 CTA
- [x] `SearchScreen` — **No Results State** Composable 구현
    - [ ] 마스코트 일러스트 (물음표 붙은 책 캐릭터)
    - [x] "직접 입력하기" 버튼 → `ManualEntryClick` 액션 발행 → `ManualBookEntryScreen` 이동
    - [ ] "Clear search history" 링크
- [ ] `RecentSearchRepository` DataStore 구현
    - [ ] `addSearch()` — 중복 제거 + 최대 10개 FIFO
    - [ ] `removeSearch()` — 개별 삭제
    - [ ] `clearAll()` — 전체 삭제
- [x] `BookGenre` enum + `subjectQuery` 매핑 _(하드코딩 → Supabase 연동으로 전환 예정)_

**Supabase genres 원격 fetch**
- [ ] Supabase Dashboard에 `genres` 테이블 생성 (스키마는 위 참조)
- [ ] 초기 장르 18개 데이터 삽입 (BISAC 기반, 4개 언어)
- [ ] `Genre` 데이터 모델에 다국어 필드 추가 (`labelEn`, `labelKo`, `labelJa`, `labelZh`)
- [ ] `GenreApiService` 구현 (Ktor로 Supabase PostgREST API 직접 호출)
- [ ] `GenreRepository` 구현 (원격 fetch → DataStore JSON 캐시 → 하드코딩 fallback)
- [ ] `SearchViewModel`에서 앱 시작 시 `GenreRepository.fetchGenres()` 호출
- [ ] 현재 언어 설정(`AppPreferences.language`)에 따라 장르 `displayName` 자동 선택
- [ ] Koin DI에 `GenreApiService`, `GenreRepository` 등록

- [x] 장르 칩 탭 시 `q=subject:{genre}` 검색 실행
- [x] `SearchViewModel` MVI 구현 (State, Action, SideEffect)
- [ ] Google Books API debounce 처리 (300ms, 2자 이상)
- [ ] 도서 추가 완료 시 `BookRepository.upsertBook()` 연동
- [ ] 도서 추가 완료 후 최근 검색어 자동 저장
- [ ] 도서 추가 완료 후 Interstitial 광고 노출 (3회마다 1번)
- [ ] Analytics 이벤트 로깅
- [ ] AdMob 배너 광고 영역 (화면 하단, 비프리미엄 사용자)
