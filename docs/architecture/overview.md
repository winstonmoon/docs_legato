# 아키텍처 개요

## 목표

- Android / iOS 공용 UI를 Compose Multiplatform으로 구현
- 공용 비즈니스 로직은 최대한 `commonMain`에 배치
- 의존성 주입은 Koin 사용
- 로컬 DB는 Room KMP 사용
- Android는 폼팩터에 따라 적응형 네비게이션 지원
- 다크모드와 다국어 기본 지원

---

## 레이어 구조

```
┌─────────────────────────────────────────┐
│              UI Layer (Compose)          │
│  Screen → ViewModel → UiState           │
├─────────────────────────────────────────┤
│              Domain Layer                │
│  UseCase → Repository Interface         │
├─────────────────────────────────────────┤
│              Data Layer                  │
│  Repository Impl → DataSource           │
├──────────────────┬──────────────────────┤
│  Remote          │  Local               │
│  Supabase        │  Room KMP            │
│  Google Books    │  DataStore           │
└──────────────────┴──────────────────────┘
```

---

## 플랫폼 분리 원칙

| 범위 | 내용 |
|---|---|
| **commonMain** | UI 화면, ViewModel, State, UseCase, Repository 인터페이스, 도메인 모델, 네비게이션 Route/Interface |
| **androidMain** | Android Navigation 호스트, 플랫폼 DI, expect/actual 구현 |
| **iosMain** | iOS Navigator 구현체, 플랫폼 DI, expect/actual 구현 |

---

## 구현 순서

### 1단계 — 기반 구조

- [ ] Koin DI 기본 설정
- [ ] `LegatoTheme` 구현 (ThemeMode: SYSTEM / LIGHT / DARK)
- [ ] 다국어 리소스 기본 구조 (ko / en / ja)
- [ ] 공용 `AppRoute`, `AppNavigator` 인터페이스 정의
- [ ] Android / iOS Navigation 호스트 분리

### 2단계 — 데이터 레이어

- [ ] Room KMP 도입 (`Book`, `ReadingSession`, `UserPreference`)
- [ ] Supabase 클라이언트 초기화
- [ ] Repository / UseCase 연결

### 3단계 — 핵심 화면

- [ ] 로그인 (Supabase Auth)
- [ ] Bottom Navigation + 5개 탭 골격
- [ ] Library, Search, Record Progress 핵심 플로우

### 4단계 — 완성

- [ ] 통계 화면
- [ ] Firebase Crashlytics / Analytics / FCM 연동
- [ ] AdMob 연동
- [ ] Android Adaptive Navigation 정교화
- [ ] DataStore 영속화

---

## 권장 기술 선택

| 항목 | 선택 | 이유 |
|---|---|---|
| **UI** | Compose Multiplatform shared UI | Android/iOS 공통 화면 |
| **DI** | Koin | KMP 공식 지원 |
| **로컬 DB** | Room KMP | Kotlin 공식, KMP 지원 |
| **네트워크** | Ktor Client | KMP 공식 지원 |
| **이미지 로딩** | Coil 3.x KMP | KMP 지원 |
| **백엔드** | Supabase | `supabase-kt` SDK |
| **Navigation (Android)** | Jetpack Navigation | 익숙한 API |
| **Navigation (iOS)** | 공용 AppNavigator 기반 Stack | 상태 기반 관리 |
