# 개발 시작 가이드

## 개발 환경 요구사항

| 항목 | 버전 |
|---|---|
| Android Studio | Meerkat (2024.3.1) 이상 |
| Xcode | 16.0 이상 |
| JDK | 17 이상 |
| Kotlin | 2.0 이상 |
| Compose Multiplatform | 1.7 이상 |
| Gradle | 8.10 이상 |

---

## 저장소 클론

```bash
git clone https://github.com/winstonmoon/legato-compose-multiplatform.git
cd legato-compose-multiplatform
```

---

## 환경 변수 설정

프로젝트 루트에 `local.properties` 파일을 생성하고 아래 값을 채운다.

```properties
# Google Books API
GOOGLE_BOOKS_API_KEY=your_api_key_here

# Supabase
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your_anon_key_here

# Android SDK (자동 생성되지 않은 경우)
sdk.dir=/Users/yourname/Library/Android/sdk
```

!!! warning "보안 주의"
    `local.properties`는 `.gitignore`에 포함되어 있다. 절대 커밋하지 않는다.
    `const val` 대신 `val`을 사용하여 ProGuard/R8 인라이닝을 방지한다.

---

## Android 빌드 & 실행

```bash
# Debug 빌드
./gradlew :composeApp:assembleDebug

# Android 에뮬레이터/디바이스 실행
./gradlew :composeApp:installDebug
```

또는 Android Studio에서 `composeApp` 모듈을 선택하고 실행.

---

## iOS 빌드 & 실행

```bash
# iOS 프레임워크 빌드
./gradlew :composeApp:linkDebugFrameworkIosSimulatorArm64

# Xcode 프로젝트 열기
open iosApp/iosApp.xcodeproj
```

Xcode에서 시뮬레이터 또는 디바이스를 선택하고 실행.

---

## 정적 분석

```bash
# Detekt 실행
./gradlew detekt

# Android Lint
./gradlew :composeApp:lint
```

---

## 브랜치 전략

| 브랜치 | 용도 |
|---|---|
| `main` | 배포 가능한 안정 버전 |
| `feat/*` | 새 기능 개발 |
| `fix/*` | 버그 수정 |
| `refactor/*` | 리팩토링 |

---

## 주요 패키지 구조

```
composeApp/src/
  commonMain/kotlin/com/winstonmoon/legato/
    core/
      designsystem/    ← 테마, 타이포그래피, 공용 컴포넌트
      model/           ← 도메인 모델 (Book, ReadingSession 등)
      data/            ← Repository 구현체, DataSource
      database/        ← Room DB, DAO, Entity
      domain/          ← UseCase
      navigation/      ← AppRoute, AppNavigator 인터페이스
      localization/    ← 언어 설정
    feature/
      home/            ← 홈 대시보드
      library/         ← 도서관/히스토리
      search/          ← 검색 & 추가
      profile/         ← 프로필 & 설정
      login/           ← 로그인
      reading/         ← 독서 기록 / 진행 상황
  androidMain/
    navigation/        ← Android Navigation 호스트
    platform/          ← Android 전용 구현
  iosMain/
    navigation/        ← iOS Navigator 구현
    platform/          ← iOS 전용 구현
```

---

## 유용한 링크

- [Stitch 디자인](https://stitch.withgoogle.com/projects/9973353658769610659){ target="_blank" }
- [Supabase 대시보드](https://supabase.com/dashboard){ target="_blank" }
- [Google Books API 문서](https://developers.google.com/books){ target="_blank" }
- [Compose Multiplatform 공식 문서](https://www.jetbrains.com/compose-multiplatform/){ target="_blank" }
- [Koin 공식 문서](https://insert-koin.io/){ target="_blank" }
