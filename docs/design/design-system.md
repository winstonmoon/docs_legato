# 디자인 시스템

[:fontawesome-solid-arrow-up-right-from-square: Stitch Design System Overview](https://stitch.withgoogle.com/projects/9973353658769610659){ target="_blank" .md-button }

---

## 디자인 토큰

| 항목 | 값 |
|---|---|
| **Primary Color** | `#137FEC` |
| **Color Mode** | Dark (기본) / Light |
| **Font** | Inter |
| **Roundness** | 8dp |
| **Saturation** | 2 |

---

## 색상 팔레트

### Light 모드

| 역할 | 색상 |
|---|---|
| Primary | `#137FEC` |
| On Primary | `#FFFFFF` |
| Background | `#FFFFFF` |
| Surface | `#F5F5F5` |
| On Surface | `#1A1A1A` |
| Error | `#B00020` |

### Dark 모드

| 역할 | 색상 |
|---|---|
| Primary | `#3D9AF0` |
| On Primary | `#FFFFFF` |
| Background | `#121212` |
| Surface | `#1E1E1E` |
| On Surface | `#E0E0E0` |
| Error | `#CF6679` |

!!! tip "독서 앱 Dark Surface 권장"
    Pure Black(`#000000`)보다 약간 누른 `#121212` ~ `#1E1E1E` 계열이 장시간 독서 시 눈의 피로를 줄인다.

---

## 타이포그래피

Inter 폰트를 사용하며, 3계층으로 단순화한다.

| 역할 | Weight | Size |
|---|---|---|
| Title | SemiBold (600) | 20sp |
| Body | Regular (400) | 16sp |
| Meta | Regular (400) | 12sp |

```kotlin
val LegatoTypography = Typography(
    titleLarge = TextStyle(
        fontFamily = InterFontFamily,
        fontWeight = FontWeight.SemiBold,
        fontSize = 20.sp,
    ),
    bodyMedium = TextStyle(
        fontFamily = InterFontFamily,
        fontWeight = FontWeight.Normal,
        fontSize = 16.sp,
    ),
    labelSmall = TextStyle(
        fontFamily = InterFontFamily,
        fontWeight = FontWeight.Normal,
        fontSize = 12.sp,
    ),
)
```

---

## 공용 컴포넌트

### LegatoTheme

```kotlin
@Composable
fun LegatoTheme(
    themeMode: ThemeMode = ThemeMode.SYSTEM,
    content: @Composable () -> Unit,
) {
    val isDark = when (themeMode) {
        ThemeMode.DARK -> true
        ThemeMode.LIGHT -> false
        ThemeMode.SYSTEM -> isSystemInDarkTheme()
    }
    MaterialTheme(
        colorScheme = if (isDark) darkColorScheme() else lightColorScheme(),
        typography = LegatoTypography,
        shapes = LegatoShapes,
        content = content,
    )
}
```

### 버튼

```kotlin
// Primary Button
LegatoPrimaryButton(text = "Add Book", onClick = { ... })

// Secondary Button
LegatoSecondaryButton(text = "Cancel", onClick = { ... })

// Text Button
LegatoTextButton(text = "수동으로 입력하기", onClick = { ... })
```

### 도서 카드

```kotlin
// 리스트 형태
BookListItem(
    book = bookUiModel,
    onClick = { onAction(LibraryAction.BookClick(it)) },
)
```

### 독서 상태 배지

```kotlin
ReadingStatusBadge(status = ReadingStatus.READING)
// → 파랑 배지 "읽는 중"

ReadingStatusBadge(status = ReadingStatus.FINISHED)
// → 초록 배지 "완독"
```

### 진행률 바

```kotlin
ReadingProgressBar(
    currentPage = 120,
    totalPages = 350,
    modifier = Modifier.fillMaxWidth(),
)
```

---

## TODO

- [ ] `LegatoTheme` 구현 (ThemeMode: SYSTEM / LIGHT / DARK)
- [ ] Primary color `#137FEC` MaterialTheme 색상 팔레트 구성 (Light / Dark)
- [ ] Inter 폰트 `composeResources/font`에 추가 + Typography 정의
- [ ] 8dp roundness `Shape` 정의
- [ ] `LegatoPrimaryButton`, `LegatoSecondaryButton`, `LegatoTextButton`
- [ ] `LegatoTextField` (에러 상태 포함)
- [ ] `BookListItem`, `BookGridItem`
- [ ] `ReadingProgressBar`
- [ ] `BottomNavigationBar` 컴포넌트
- [ ] `ReadingStatusBadge` (색상 구분)
- [ ] Android 시스템 다크모드 감지 → `ThemeMode.SYSTEM` 반영
- [ ] iOS 시스템 appearance 감지 → `ThemeMode.SYSTEM` 반영 (expect/actual)
