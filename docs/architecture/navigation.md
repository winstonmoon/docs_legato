# 네비게이션 전략

## 기본 원칙

화면과 상태는 공유하고, 네비게이션 호스트는 플랫폼별로 분리한다.

| 범위 | 공용 (`commonMain`) | Android | iOS |
|---|---|---|---|
| **Route 정의** | ✅ `AppRoute` | — | — |
| **Navigator 인터페이스** | ✅ `AppNavigator` | — | — |
| **화면 Composable** | ✅ | — | — |
| **Navigation 호스트** | — | ✅ NavHost | ✅ Stack Navigator |
| **탭 표현** | — | ✅ Bottom / Rail / Drawer | ✅ Compose TabView |

---

## 공용 Route

```kotlin
sealed interface AppRoute {
    // Bottom Nav 탭
    data object Home : AppRoute
    data object Library : AppRoute
    data object Search : AppRoute
    data object Profile : AppRoute

    // 상세 화면
    data class RecordProgress(val bookId: String) : AppRoute
    data class LogReading(val bookId: String) : AppRoute
    data class ManualEntry : AppRoute

    // Auth
    data object Login : AppRoute
}
```

---

## 공용 Navigator 인터페이스

```kotlin
interface AppNavigator {
    val currentRoute: StateFlow<AppRoute>
    fun navigate(route: AppRoute)
    fun back()
    fun navigateAndClearStack(route: AppRoute) // 로그인 완료 후 사용
}
```

---

## Android 네비게이션

```kotlin
// NavHost + BottomNavigation (Compact)
@Composable
fun AndroidNavHost(navigator: AppNavigator) {
    val navController = rememberNavController()
    NavHost(navController, startDestination = "home") {
        composable("home") { HomeScreen() }
        composable("library") { LibraryScreen() }
        composable("search") { SearchScreen() }
        composable("profile") { ProfileScreen() }
        composable("record/{bookId}") { RecordProgressScreen() }
        composable("log/{bookId}") { LogReadingScreen() }
        composable("manual") { ManualBookEntryScreen() }
    }
}
```

---

## Android Adaptive Navigation

```kotlin
val windowSizeClass = calculateWindowSizeClass()

when (windowSizeClass.widthSizeClass) {
    WindowWidthSizeClass.Compact ->
        LegatoBottomBar(tabs, currentRoute, onTabSelected)

    WindowWidthSizeClass.Medium ->
        LegatoNavigationRail(tabs, currentRoute, onTabSelected)

    WindowWidthSizeClass.Expanded ->
        LegatoPermanentDrawer(tabs, currentRoute, onTabSelected)
}
```

---

## iOS 네비게이션

공용 `AppNavigator` 구현체를 사용한 Stack 기반 호스트.

```kotlin
// iosMain
class IosAppNavigator : AppNavigator {
    private val _currentRoute = MutableStateFlow<AppRoute>(AppRoute.Home)
    override val currentRoute: StateFlow<AppRoute> = _currentRoute

    private val backStack = mutableListOf<AppRoute>()

    override fun navigate(route: AppRoute) {
        backStack.add(_currentRoute.value)
        _currentRoute.value = route
    }

    override fun back() {
        if (backStack.isNotEmpty()) {
            _currentRoute.value = backStack.removeLast()
        }
    }
}
```

---

## 탭 정의 (공용)

```kotlin
data class MainTab(
    val route: AppRoute,
    val labelRes: StringResource,
    val icon: ImageVector,
)

val mainTabs = listOf(
    MainTab(AppRoute.Home, Res.string.tab_home, Icons.Default.Home),
    MainTab(AppRoute.Library, Res.string.tab_library, Icons.Default.Book),
    MainTab(AppRoute.Search, Res.string.tab_search, Icons.Default.Search),
    MainTab(AppRoute.Profile, Res.string.tab_profile, Icons.Default.Person),
)
```
