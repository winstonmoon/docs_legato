# 유저 플로우

## 플로우 1: 새 도서 추가 (검색)

```mermaid
flowchart LR
    A[홈] --> B[검색 탭]
    B --> C[검색어 입력]
    C --> D{결과?}
    D -- 있음 --> E[결과 선택]
    E --> F[Add to Library]
    F --> G[홈 반영]
    D -- 없음 --> H[수동 입력]
    H --> I[폼 작성]
    I --> G
```

---

## 플로우 2: 독서 진행 기록

```mermaid
flowchart LR
    A[홈 / 라이브러리] --> B[도서 선택]
    B --> C[Record Progress 화면]
    C --> D[시작~끝 페이지 입력]
    D --> E{완독?}
    E -- 예 --> F[완독 상태로 전환]
    F --> G[축하 UI 표시]
    G --> H[저장 완료]
    E -- 아니오 --> H
    H --> I[홈/라이브러리 업데이트]
```

---

## 플로우 3: 독서 히스토리 확인

```mermaid
flowchart LR
    A[Library 탭] --> B[상태 필터 선택]
    B --> C[도서 목록 조회]
    C --> D[도서 클릭]
    D --> E[세션별 기록 확인]
```

---

## 플로우 4: 인증 흐름

```mermaid
flowchart TD
    A[앱 실행] --> B{세션 있음?}
    B -- 예 --> C[홈 대시보드]
    B -- 아니오 --> D[로그인 화면]
    D --> E{로그인 방법}
    E -- Google --> F[Supabase OAuth]
    E -- Apple --> G[Supabase OAuth]
    E -- 이메일 --> H[이메일/비밀번호]
    F --> I{성공?}
    G --> I
    H --> I
    I -- 예 --> C
    I -- 아니오 --> J[에러 메시지]
    J --> D
```

---

## 플로우 5: 설정 변경

```mermaid
flowchart LR
    A[Profile 탭] --> B[설정 선택]
    B --> C{항목}
    C -- 테마 --> D[ThemeMode 변경]
    D --> E[DataStore 저장]
    E --> F[앱 전체 테마 적용]
    C -- 언어 --> G[Language 변경]
    G --> E
    C -- 로그아웃 --> H[Supabase signOut]
    H --> I[로그인 화면으로 이동]
```

---

## 화면 전환 맵

| 출발 | 목적지 | 트리거 |
|---|---|---|
| Splash | Login | 세션 없음 |
| Splash | Home | 세션 있음 |
| Login | Home | 로그인 성공 |
| Home | Record Progress | "계속 읽기" 클릭 |
| Library | Record Progress | 도서 클릭 |
| Search | Manual Entry | "수동으로 입력하기" 클릭 |
| Search | Library | 도서 추가 성공 |
| Profile | Login | 로그아웃 |
