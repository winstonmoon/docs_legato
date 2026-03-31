# ISBN 바코드 스캐너 (Barcode Scanner)

[:fontawesome-solid-arrow-up-right-from-square: Stitch에서 보기](https://stitch.withgoogle.com/projects/9973353658769610659){ target="_blank" .md-button }

---

## 개요

ISBN 바코드를 카메라로 스캔하여 Google Books API 검색으로 자동 연동하는 화면.
검색 화면의 `barcode_scanner` 아이콘 탭 시 진입.

---

## UI 구성

| 구성 요소 | 설명 |
|---|---|
| 카메라 뷰파인더 | 전체 화면 카메라 프리뷰 |
| 스캔 가이드 | 중앙 직사각형 가이드 오버레이 |
| 닫기 버튼 | 화면 좌상단 — 스캐너 종료 |
| 안내 텍스트 | "책 뒷면의 바코드를 스캔하세요" |
| 스캔 성공 피드백 | 바코드 인식 시 진동 + 검색 화면으로 자동 이동 |

---

## 동작 플로우

```
SearchScreen barcode_scanner 아이콘 탭
→ BarcodeScannerScreen
→ ISBN 스캔 성공 → SearchScreen (ISBN 자동 검색 실행)
→ 닫기 → SearchScreen (검색어 유지)
```

---

## 구현 방식

`IsbnBarcodeScanner` expect/actual 구현:
- **Android**: ML Kit Barcode Scanning
- **iOS**: AVFoundation + VisionKit

---

## TODO

- [ ] `BarcodeScannerScreen` Composable 구현
- [ ] 카메라 권한 요청 처리 (expect/actual)
- [ ] `IsbnBarcodeScanner` expect/actual 구현
    - [ ] Android: ML Kit Barcode Scanning 연동
    - [ ] iOS: AVFoundation / VisionKit 연동
- [ ] 스캔 성공 시 ISBN → `SearchScreen` 자동 검색 연동
- [ ] `AppRoute.BarcodeScanner` 라우트 추가
- [ ] 닫기 버튼 → `SearchScreen`으로 복귀
