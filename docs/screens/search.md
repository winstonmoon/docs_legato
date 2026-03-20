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

> Stitch 디자인: **Search - Initial (Light)** / **Search - Initial (Dark)**

| 구성 요소 | 설명 |
|---|---|
| 검색 바 | 탭 시 검색 입력 모드로 전환 |
| Hero 섹션 | 마스코트 일러스트 + "Find your next story" 헤딩 + 서브 텍스트 |
| Recent Searches | 최근 검색어 칩 목록 (칩별 ✕ 버튼 + "Clear all" 버튼) |
| Explore Genres | 장르 필터 칩 가로 스크롤 목록 |
| Popular This Week | 이번 주 인기 도서 카드 (표지, 제목, 저자) |
| 하단 내비게이션 | Home · Search · Library · Profile |

### Stitch 스크린샷

=== "Light"
    ![Search Initial Light](https://lh3.googleusercontent.com/aida/ADBb0ugB26zYwtKZkOs_shLBVAfE2NTvIcWSXGDyhC4dl8gE7x3Cc3i-La83sVOvsEulET8LjQoiweoTmPxgB2Mq4d2FO93HA5CdIo1ntMLFsn5NGzijqdmx3WzULJHT5c-XAyz7szbcOr84RrwUZu2CJFWteA7qhVI6z8bgRMYUoZEvphyQ-3rz0RoplLIZaJ3Hv3yk1SBxvl2THIiNdl_LThFC_xSQ4a3_eTWqKiwZ_fpoQzWtHNu_DvTDnyQ)

=== "Dark"
    ![Search Initial Dark](https://lh3.googleusercontent.com/aida/ADBb0ugb-hbrDlxq7KzLGlo_6Z5jU9RmNt-H-ZLoUVWGxU90vC2YiI_8JLtdTgfkuq2PsN7NN2kUU7h5z9s9Z7DceB6hIoXMsiJUCjUc9dseItEPkkanCJ7ENhPBqeaZ4fcU9dL_Ip8H8V1zNNjw1QyZ1q9v_v3UWaIbbUPqzfnAly50Fx0AyvHMwRNkUwElHhykXoBmgWDonbQWJWYUKigP0m2aTbGB9vYM30aUUbj9xuS80aYrcpm4UHxU2V0)

---

## Search Results UI 구성

> Stitch 디자인: **Search and Add Book Updated**

| 구성 요소 | 설명 |
|---|---|
| 검색 입력 필드 | 뒤로가기 버튼 + "Search Books" 타이틀 |
| 필터 바 | Genre · Author · Rating · Year 드롭다운 칩 (가로 스크롤) |
| 검색 결과 리스트 | 책 표지, 제목, 저자, 별점(★), 출판 연도, Add/Added 버튼 |
| 하단 내비게이션 | Home · Search · Library · Profile |

### Stitch 스크린샷

![Search and Add Updated](https://lh3.googleusercontent.com/aida/ADBb0ugnOlahDZq30aIqFzZsRfnYSzuXOxV6Ly-WB-Q4KrJC7CO28U_9KUhyRAkmsqAR35jI8seiIxYBpLM7zIdl0NBbiXcgzdTb3ggzQpnj4q0q0RYWrt6uRnfa5VLx1NVYrZwJxjeY3F-IcQL2m53WYxUzCFNkoW809dFdokXUMiZFHyzC9PJWspxrKYBE1lFooick0aVmHMInSJCvxys7lFqQo_JFrY3WGr6rChJNy1I0NtNxgrH9YzRhog)

---

## Explore Genres — 장르 칩

### Google Books API 장르 지원 여부

!!! info "Google Books API에는 전용 장르 목록 엔드포인트가 없습니다"
    `/genres` 또는 `/categories` 전용 API는 제공되지 않습니다.
    대신 **`subject:` 필터**를 이용해 특정 장르의 책을 검색합니다.

    ```
    GET https://www.googleapis.com/books/v1/volumes
      ?q=subject:Fiction
      &orderBy=relevance
      &maxResults=20
    ```

    각 책의 응답에는 `volumeInfo.categories[]` 필드가 포함되나, 앱 UI에 보여줄 장르 목록 자체는 **앱 내 하드코딩**이 현실적입니다.

### 장르 목록 (하드코딩 + Google Books subject 매핑)

| UI 칩 레이블 | Google Books `subject` 쿼리 |
|---|---|
| Fiction | `subject:Fiction` |
| Philosophy | `subject:Philosophy` |
| Sci-Fi | `subject:Science+Fiction` |
| Classic Literature | `subject:Classics` |
| Mystery | `subject:Mystery` |
| Poetry | `subject:Poetry` |
| History | `subject:History` |
| Biography | `subject:Biography` |

```kotlin
enum class BookGenre(val label: String, val subjectQuery: String) {
    FICTION("Fiction", "subject:Fiction"),
    PHILOSOPHY("Philosophy", "subject:Philosophy"),
    SCI_FI("Sci-Fi", "subject:Science+Fiction"),
    CLASSICS("Classic Literature", "subject:Classics"),
    MYSTERY("Mystery", "subject:Mystery"),
    POETRY("Poetry", "subject:Poetry"),
    HISTORY("History", "subject:History"),
    BIOGRAPHY("Biography", "subject:Biography"),
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

- [ ] `SearchScreen` — Initial State Composable 구현 (Light / Dark)
    - [ ] Hero 섹션 (마스코트 이미지 + "Find your next story" 텍스트)
    - [ ] Recent Searches 섹션 (칩 + ✕ 버튼 + "Clear all")
    - [ ] Explore Genres 섹션 (가로 스크롤 칩 목록, `BookGenre` enum 기반)
    - [ ] Popular This Week 섹션 (Google Books API `orderBy=relevance` 인기 도서)
- [ ] `SearchScreen` — Results State Composable 구현
    - [ ] 필터 바 (Genre · Author · Rating · Year 드롭다운)
    - [ ] 검색 결과 아이템 (표지, 제목, 저자, 별점, 출판 연도, Add/Added 버튼)
- [ ] `RecentSearchRepository` DataStore 구현
    - [ ] `addSearch()` — 중복 제거 + 최대 10개 FIFO
    - [ ] `removeSearch()` — 개별 삭제
    - [ ] `clearAll()` — 전체 삭제
- [ ] `BookGenre` enum + `subjectQuery` 매핑
- [ ] 장르 칩 탭 시 `q=subject:{genre}` 검색 실행
- [ ] `SearchViewModel` MVI 구현 (State, Action, SideEffect)
- [ ] Google Books API debounce 처리 (300ms, 2자 이상)
- [ ] 도서 추가 완료 시 `BookRepository.upsertBook()` 연동
- [ ] 도서 추가 완료 후 최근 검색어 자동 저장
- [ ] 도서 추가 완료 후 Interstitial 광고 노출 (3회마다 1번)
- [ ] Analytics 이벤트 로깅
- [ ] AdMob 배너 광고 영역 (화면 하단, 비프리미엄 사용자)
- [ ] No Results 상태: "수동으로 입력하기" 버튼 → Manual Book Entry 이동
