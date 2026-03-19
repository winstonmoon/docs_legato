# Legato — Connected Reading

<div align="center">
  <h3>📚 독서 기록 & 관리 앱 개발 문서</h3>
  <p>Compose Multiplatform (Android / iOS)</p>
</div>

---

## 프로젝트 개요

**Legato**는 사용자가 읽고 있는 책을 추적하고, 독서 히스토리를 관리하며, 독서 진행 상황을 기록할 수 있는 크로스플랫폼 독서 기록 앱이다.

| 항목 | 내용 |
|---|---|
| **앱 이름** | Legato — Connected Reading |
| **플랫폼** | Android / iOS (Compose Multiplatform) |
| **아키텍처** | MVI (Model-View-Intent) |
| **UI 프레임워크** | Jetpack Compose / Compose Multiplatform |
| **DI** | Koin |
| **로컬 DB** | Room KMP |
| **백엔드** | Supabase (PostgreSQL + Auth + Storage) |
| **디자인** | [Stitch 보기 :fontawesome-solid-arrow-up-right-from-square:](https://stitch.withgoogle.com/projects/9973353658769610659){ target="_blank" } |

---

## 핵심 가치

- **간편한 도서 검색** — Google Books API 연동으로 빠른 도서 추가
- **세션 단위 독서 기록** — 시작/끝 페이지 및 메모 기록
- **독서 히스토리 & 통계** — 주간/월간 독서량 시각화
- **크로스플랫폼** — Android와 iOS에서 동일한 UI 경험

---

## 화면 구성

```
┌── 로그인 (Login)
└── 메인 (Bottom Navigation)
    ├── 🏠 홈 대시보드 (Home Dashboard)
    ├── 📚 도서관 / 히스토리 (Library)
    ├── 🔍 검색 & 추가 (Search & Add)
    └── 👤 프로필 & 설정 (Profile)
            ├── 독서 기록 (Log Reading)
            └── 진행 상황 기록 (Record Progress)
```

---

## 빠른 링크

<div class="grid cards" markdown>

-   :material-palette:{ .lg .middle } **디자인 (Stitch)**

    ---

    Stitch에서 전체 화면 디자인을 확인할 수 있다.

    [:octicons-arrow-right-24: Stitch 열기](https://stitch.withgoogle.com/projects/9973353658769610659){ target="_blank" }

-   :material-layers:{ .lg .middle } **아키텍처**

    ---

    모듈 구조, 네비게이션, 데이터 모델, DI 설계

    [:octicons-arrow-right-24: 아키텍처 보기](architecture/overview.md)

-   :material-monitor-cellphone:{ .lg .middle } **화면 스펙**

    ---

    화면별 UI 구성, 기능, 구현 TODO

    [:octicons-arrow-right-24: 화면 목록 보기](screens/index.md)

-   :material-server:{ .lg .middle } **백엔드 & 서비스**

    ---

    Supabase, Firebase, AdMob 연동 가이드

    [:octicons-arrow-right-24: 백엔드 보기](backend/supabase.md)

</div>

---

## 기술 스택 요약

| 분류 | 기술 |
|---|---|
| **UI** | Compose Multiplatform |
| **아키텍처** | MVI |
| **DI** | Koin |
| **로컬 DB** | Room KMP |
| **네트워크** | Ktor Client |
| **이미지 로딩** | Coil KMP |
| **백엔드** | Supabase (Auth, PostgreSQL, Storage) |
| **도서 검색** | Google Books API |
| **크래시 리포팅** | Firebase Crashlytics |
| **앱 분석** | Firebase Analytics |
| **푸시 알림** | Firebase Cloud Messaging (FCM) |
| **광고** | Google AdMob |
| **정적 분석** | Detekt + Android Lint |
