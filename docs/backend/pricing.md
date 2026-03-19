# 요금 & 구현 난이도

## 요금

| 서비스 | 무료 플랜 | 유료 플랜 |
|---|---|---|
| **Supabase** | 500MB DB, 50K MAU, 1GB 스토리지 | Pro: $25/월 |
| **Firebase Crashlytics** | 완전 무료 (한도 없음) | — |
| **Firebase Analytics** | 완전 무료 (이벤트 500종) | — |
| **Firebase Cloud Messaging** | 완전 무료 (발송 수 제한 없음) | — |
| **Google AdMob** | 무료 사용 | 수익 ~68% 수령 (Google ~32% 수수료) |

!!! warning "Supabase Free 플랜 주의"
    7일 비활성 시 프로젝트 자동 일시정지.
    실사용 서비스에는 **Pro ($25/월)** 필요.

---

## 구현 난이도 (Compose Multiplatform 기준)

| 서비스 | 난이도 | 이유 |
|---|---|---|
| **Supabase** | ⭐ 쉬움 | 전용 KMP SDK (`supabase-kt`) 있음 |
| **Firebase Crashlytics** | ⭐⭐ 보통 | GitLiveApp `firebase-kotlin-sdk`로 해결 |
| **Firebase Analytics** | ⭐⭐ 보통 | GitLiveApp `firebase-kotlin-sdk`로 해결 |
| **Firebase Cloud Messaging** | ⭐⭐ 보통 | `KMPNotifier` 라이브러리로 해결 |
| **Google AdMob** | ⭐⭐⭐ 어려움 | 공식 KMP 지원 없음, expect/actual 직접 구현 필요 |

---

## 핵심 라이브러리

| 서비스 | 라이브러리 | 저장소 |
|---|---|---|
| Supabase | `io.github.jan-tennert.supabase:postgrest-kt` | [GitHub](https://github.com/supabase-community/supabase-kt) |
| Firebase (통합) | `dev.gitlive:firebase-*` | [GitHub](https://github.com/GitLiveApp/firebase-kotlin-sdk) |
| FCM 대안 | `io.github.mirzemehdi:kmpnotifier` | [GitHub](https://github.com/mirzemehdi/KMPNotifier) |
| AdMob | expect/actual 직접 구현 또는 LexiLabs `basic-ads` | — |

---

## 구현 우선순위 추천

```
1순위 (MVP)
  └── Supabase (Auth + DB)         ← 쉬움, 핵심 기능
  └── Firebase Crashlytics         ← 쉬움, 안정성 필수

2순위 (런칭 전)
  └── Firebase Analytics           ← 보통, 사용자 데이터 수집
  └── FCM                          ← 보통, 알림 기능

3순위 (수익화 시)
  └── AdMob                        ← 어려움, 수익화 필요 시 구현
```

---

## 기타 참고 사항

- Firebase iOS는 dSYM 파일을 Xcode 빌드 페이즈에서 별도 업로드해야 Kotlin/Native 스택 트레이스가 정상 출력된다
- FCM 발송 서버는 Supabase Edge Function으로 처리 가능 (별도 서버 불필요)
- AdMob은 GDPR 동의 SDK (UMP) 추가 연동 필수
- Supabase Storage는 Free 플랜에서 1GB 제공 (표지 이미지 저장에 충분)
