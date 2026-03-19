# 상태 관리 (MVI)

## MVI 패턴 구조

```
┌─────────┐   Action   ┌────────────┐   State    ┌────────┐
│   UI    │ ─────────→ │ ViewModel  │ ─────────→ │   UI   │
│ Screen  │            │  Reducer   │            │ render │
└─────────┘            └─────┬──────┘            └────────┘
                             │ SideEffect
                             ↓
                      Navigation / Toast / etc.
```

---

## 표준 구조

각 화면은 동일한 패턴으로 구성된다.

```kotlin
// 1. UiState — 화면에 필요한 모든 상태
data class ExampleUiState(
    val isLoading: Boolean = false,
    val data: List<Item> = emptyList(),
    val error: String? = null,
)

// 2. Action — 사용자 입력 이벤트
sealed interface ExampleAction {
    data object RefreshClick : ExampleAction
    data class ItemClick(val id: String) : ExampleAction
}

// 3. SideEffect — one-shot 이벤트 (네비게이션, 토스트)
sealed interface ExampleSideEffect {
    data class NavigateToDetail(val id: String) : ExampleSideEffect
    data class ShowError(val message: String) : ExampleSideEffect
}
```

---

## ViewModel 구현 패턴

```kotlin
class ExampleViewModel(
    private val repository: ExampleRepository,
) : ViewModel() {

    private val _uiState = MutableStateFlow(ExampleUiState())
    val uiState: StateFlow<ExampleUiState> = _uiState.asStateFlow()

    private val _sideEffect = Channel<ExampleSideEffect>(Channel.BUFFERED)
    val sideEffect: Flow<ExampleSideEffect> = _sideEffect.receiveAsFlow()

    fun onAction(action: ExampleAction) {
        when (action) {
            is ExampleAction.RefreshClick -> loadData()
            is ExampleAction.ItemClick -> {
                viewModelScope.launch {
                    _sideEffect.send(ExampleSideEffect.NavigateToDetail(action.id))
                }
            }
        }
    }

    private fun loadData() {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true) }
            repository.observeItems()
                .catch { e ->
                    _uiState.update { it.copy(isLoading = false, error = e.message) }
                }
                .collect { items ->
                    _uiState.update { it.copy(isLoading = false, data = items) }
                }
        }
    }
}
```

---

## Screen에서 사용하는 방법

```kotlin
@Composable
fun ExampleScreen(
    viewModel: ExampleViewModel = koinViewModel(),
    navigator: AppNavigator,
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    LaunchedEffect(Unit) {
        viewModel.sideEffect.collect { effect ->
            when (effect) {
                is ExampleSideEffect.NavigateToDetail ->
                    navigator.navigate(AppRoute.Detail(effect.id))
                is ExampleSideEffect.ShowError ->
                    // Snackbar 표시
            }
        }
    }

    ExampleContent(
        uiState = uiState,
        onAction = viewModel::onAction,
    )
}
```

---

## 규칙

| 규칙 | 내용 |
|---|---|
| UI는 `UiState`만 본다 | ViewModel 내부 상태 직접 접근 금지 |
| 사용자 입력은 `Action`으로 | 직접 ViewModel 메서드 호출 지양 |
| One-shot 이벤트는 `SideEffect` | `SharedFlow` 또는 `Channel` 사용 |
| Repository는 `Flow` 기반 | `suspend` + `collect` 패턴 유지 |
| UI 모델은 별도 분리 | `BookItemUiModel` ≠ `Book` 도메인 모델 |
