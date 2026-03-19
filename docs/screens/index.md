# 화면 목록

[:fontawesome-solid-arrow-up-right-from-square: Stitch에서 전체 디자인 보기](https://stitch.withgoogle.com/projects/9973353658769610659){ target="_blank" .md-button .md-button--primary }

---

## 화면 구성도

```
┌── 로그인 (Login)
└── 메인 (Bottom Navigation)
    ├── 🏠 홈 대시보드 (Home Dashboard)
    ├── 📚 도서관 / 히스토리 (Library History)
    ├── 🔍 검색 & 추가 (Search & Add Book)
    │       └── 수동 도서 입력 (Manual Book Entry)
    └── 👤 프로필 & 설정 (Profile & Settings)

도서 클릭 시:
    ├── 독서 기록 (Log Reading)
    └── 독서 진행 상황 기록 (Record Reading Progress)
```

---

## 화면 목록 & 구현 상태

| 화면 | Stitch | 다크 | 라이트 | 구현 |
|---|---|---|---|---|
| [로그인](login.md) | ✅ | ✅ | ✅ | ⬜ |
| [홈 대시보드](home.md) | ✅ | ✅ | ✅ | ⬜ |
| [도서관 / 히스토리](library.md) | ✅ | ✅ | ✅ | ⬜ |
| [검색 & 추가](search.md) | ✅ | ✅ | ✅ | ⬜ |
| [수동 도서 입력](manual-entry.md) | ✅ | ✅ | ✅ | ⬜ |
| [독서 기록](log-reading.md) | ✅ | ⬜ | ✅ | ⬜ |
| [독서 진행 상황 기록](record-progress.md) | ✅ | ⬜ | ✅ | ⬜ |
| [프로필 & 설정](profile.md) | ✅ | ✅ | ✅ | ⬜ |

---

## 하단 내비게이션 탭

| 탭 | 아이콘 | 화면 |
|---|---|---|
| Home | `home` | 홈 대시보드 |
| Library | `book` | 도서관 / 히스토리 |
| Search | `search` | 검색 & 추가 |
| Profile | `person` | 프로필 & 설정 |
