# 기찬뷰어 (Gichan Viewer) — PRD 설계 문서

## 1. 개요

| 항목 | 내용 |
|------|------|
| 프로젝트명 | 기찬뷰어 (Gichan Viewer) |
| 유형 | PWA (Progressive Web App) — 읽기 전용 |
| 상위 프로젝트 | LifeRecord (기찬다이어리 WPF) |
| 호스팅 | GitHub Pages (무료) |
| 대상 기기 | 스마트폰/태블릿 (세로 모드 기본) |
| 데이터 소스 | 기찬다이어리에서 생성한 Excel (.xlsx) |
| 빌드 도구 | 없음 (Vanilla HTML/CSS/JS) |

### 1.1 핵심 목표
- 기찬다이어리 데이터를 스마트폰에서 조회/분석
- WPF 앱과 동일한 색상/UX 경험 유지
- 오프라인 동작 (마지막 Import 데이터 캐시)
- 서버 비용 0원, 앱스토어 등록 불필요

---

## 2. 기술 스택

| 항목 | 기술 | 비고 |
|------|------|------|
| 마크업 | HTML5 | 단일 페이지 (SPA) |
| 스타일 | CSS3 | CSS 변수로 테마 전환 |
| 스크립트 | Vanilla JavaScript (ES Modules) | 프레임워크 없음 |
| Excel 파싱 | SheetJS (xlsx.mini.min.js) | 브라우저 로컬 파싱 |
| 차트 | Chart.js 4.x | Line, Bar, Doughnut |
| 오프라인 | Service Worker | 앱 자원 캐시 |
| 데이터 캐시 | IndexedDB | Import 데이터 보존 |
| 설정 저장 | LocalStorage | 테마, 페이지 건수 등 |

---

## 3. 폴더 구조

```
ExtendPrj/
├── docs/
│   ├── 요구사항문서.txt
│   ├── 작업 메모 기록.txt
│   └── superpowers/specs/
│       └── 2026-03-30-gichan-viewer-design.md  (본 문서)
├── design/                          # GUI 디자인 자료
├── src/gichan-viewer/               # PWA 소스 (= GitHub Pages 배포 루트)
│   ├── index.html                   # 단일 HTML (SPA)
│   ├── manifest.json                # PWA 매니페스트
│   ├── sw.js                        # Service Worker
│   ├── css/
│   │   └── style.css                # 테마 변수 + 모바일 레이아웃
│   ├── js/
│   │   ├── app.js                   # 앱 초기화, 탭 전환, 테마
│   │   ├── excel-parser.js          # SheetJS → BabyEvent[] 변환
│   │   ├── storage.js               # IndexedDB 캐시
│   │   ├── calc.js                  # 계산 로직 (WPF CalculationService 포팅)
│   │   ├── dashboard.js             # 현황탭 렌더링
│   │   ├── browse.js                # 조회탭 필터/검색/페이지네이션
│   │   ├── report.js                # 리포트탭 차트 + 요약
│   │   └── settings.js              # 설정탭
│   ├── lib/
│   │   ├── xlsx.mini.min.js         # SheetJS 번들
│   │   └── chart.min.js             # Chart.js 번들
│   └── icons/
│       ├── icon-192.png             # PWA 아이콘
│       └── icon-512.png
└── CLAUDE.md
```

---

## 4. 데이터 모델

### 4.1 Excel 컬럼 매핑 (LifeRecord 동일)

| 컬럼 | 이름 | JS 프로퍼티 | 타입 |
|------|------|------------|------|
| A | 일차 | dayNumber | number? |
| B | 날짜 | date | Date |
| C | 시간 | time | string? |
| D | 구분 | category | string |
| E | 세부내용 | detail | string |
| F | 양(ml) | amount | string |
| G | 비고 | note | string |
| H | 수유텀 | feedingInterval | string? |
| I | 다음예상 | nextExpected | string? |
| J | 일일수유량 | dailyFeedTotal | number? |
| K | 분유제품 | formulaProduct | string? |
| L | 분유량(ml) | formulaAmount | number? |
| M | 모유수유량(ml) | breastfeedAmount | number? |
| N | 수유횟수 | feedingCount | number? |
| O | 소변여부 | hasUrine | boolean? |
| P | 대변여부 | hasStool | boolean? |
| Q | 직후인지 | immediateNotice | boolean? |

### 4.2 카테고리 매핑

```javascript
const CATEGORIES = ['수유', '배변', '위생관리', '신체측정', '건강관리', '기타'];
// 레거시: '위생관리' ← '위생', '통증' ← '건강관리'
```

---

## 5. 화면 설계

### 5.1 전체 레이아웃

```
┌─────────────────────────┐
│  기찬뷰어  [🌙/☀️ 테마]  │  ← 헤더 (고정)
├─────────────────────────┤
│                         │
│     [탭 컨텐츠 영역]      │  ← 스크롤 가능
│                         │
├─────────────────────────┤
│ 현황 │ 조회 │ 리포트 │ 설정 │  ← 하단 탭바 (고정)
└─────────────────────────┘
```

### 5.2 현황탭

카드 세로 1열 배치:
- **수유 카드**: 최근 시간, 경과시간, 세부내용(분할 N회), 오늘 누적(Nml/N회), 평균 수유텀
- **배변 카드**: 소변 경과, 대변 경과, 오늘 소변/대변 횟수
- **위생 카드**: 세안/목욕/손발톱 각 최근 날짜+경과시간
- **신체 카드**: 키/몸무게/머리둘레 최근 기록
- **건강 카드**: 최근 건강 기록
- **기타 카드**: 최근 기타 기록

### 5.3 조회탭

- 상단: 날짜 범위 (시작~종료) + 검색 입력
- 중단: 카테고리 필터 체크박스 6개 (가로 배치, 전체 기본 체크)
- 하단: 카드형 리스트 (DataGrid 대신 카드 — 모바일 가독성)
  - 각 카드: 날짜+시간 / 카테고리(색상) / 세부내용(색상) / 양
- 페이지네이션: 이전/다음 버튼 + 페이지 표시

### 5.4 리포트탭

- 기간 선택 버튼 (7일/14일/30일/전체)
- 요약 카드 가로 스크롤 (5개)
- 6종 차트 세로 배치 (각 차트 카드)

### 5.5 설정탭

- Excel 파일 가져오기 버튼 (큰 버튼)
- 마지막 Import 정보 (날짜, 건수)
- 페이지당 표시 건수 (10/20/30/50 라디오)
- 아기 이름 입력
- 테마 전환 (Dark/Light 토글)
- 앱 정보 (버전, 상위 프로젝트 링크)

---

## 6. 색상 팔레트

WPF Colors.xaml / LightColors.xaml → CSS 변수 변환:

### Dark 테마
```css
:root[data-theme="dark"] {
  --bg-dark: #242A3C;
  --bg-card: #2A3248;
  --bg-sidebar: #1E2438;
  --bg-input: #1A2030;
  --text-light: #C8D0DC;
  --text-mid: #8090A8;
  --border: #2A3050;
  --cat-feed: #1E8070;
  --cat-bowel: #B88020;
  --cat-hygiene: #305898;
  --cat-body: #704890;
  --cat-health: #A04040;
  --cat-etc: #506068;
  --cat-urine: #D4B840;
  --cat-stool: #8B4513;
}
```

### Light 테마
```css
:root[data-theme="light"] {
  --bg-dark: #B8BEC8;
  --bg-card: #C8CDD6;
  --bg-sidebar: #B0B6C0;
  --bg-input: #C0C5D0;
  --text-light: #1A1F2E;
  --text-mid: #5A6A8A;
  --border: #9098A8;
  --cat-feed: #167060;
  --cat-bowel: #9A6A10;
  --cat-hygiene: #285088;
  --cat-body: #604080;
  --cat-health: #903030;
  --cat-etc: #405058;
  --cat-urine: #B8A030;
  --cat-stool: #783C10;
}
```

---

## 7. 계산 로직 (WPF CalculationService 포팅)

JS로 포팅할 함수 목록:

| WPF 메서드 | JS 함수 | 용도 |
|-----------|---------|------|
| GetDailyFeedTotal | getDailyFeedTotal(events, date) | 일일 수유량 합계 |
| GetDailyFeedCount | getDailyFeedCount(events, date) | 일일 수유 횟수 |
| GetDailySummary | getDailySummary(events, date) | 일일 소변/대변 횟수 |
| GetLastUrineElapsed | getLastUrineElapsed(events) | 마지막 소변 경과 |
| GetLastStoolElapsed | getLastStoolElapsed(events) | 마지막 대변 경과 |
| GetAverageFeedingInterval | getAvgFeedingInterval(events, count) | 평균 수유텀 |
| CalculateDayNumber | calcDayNumber(date, birthDate) | 일차 계산 |

---

## 8. 데이터 흐름

```
사용자 조작                    처리                          저장
─────────────────────────────────────────────────────────────
설정탭 "파일 가져오기"
    → <input type="file">
    → SheetJS 파싱
    → BabyEvent[] 배열 생성
    → IndexedDB 저장 ──────────────────→ IndexedDB
    → 현황/조회/리포트 자동 갱신

앱 재실행
    → IndexedDB에서 캐시 데이터 로드 ←── IndexedDB
    → 현황/조회/리포트 표시

설정 변경
    → LocalStorage 저장 ────────────────→ LocalStorage
    → UI 즉시 반영
```

---

## 9. PWA 구성

### manifest.json
```json
{
  "name": "기찬뷰어",
  "short_name": "기찬뷰어",
  "start_url": "./index.html",
  "display": "standalone",
  "background_color": "#242A3C",
  "theme_color": "#1E2438",
  "icons": [
    { "src": "icons/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "icons/icon-512.png", "sizes": "512x512", "type": "image/png" }
  ]
}
```

### Service Worker 전략
- Install: 앱 자원(HTML/CSS/JS/lib) 캐시
- Fetch: Cache First → Network Fallback
- 데이터(IndexedDB)는 Service Worker와 무관하게 별도 관리

---

## 10. Excel 호환성

- 기존 컬럼 A~J: 필수 지원
- 확장 컬럼 K~Q: 있으면 사용, 없으면 빈값
- 카테고리 레거시 매핑: "위생관리"←"위생", "건강관리"←"통증"
- 시간 형식: "HH:mm", "HH:mm:ss", Excel 시간값(소수) 모두 파싱
- 날짜 형식: Excel 시리얼 넘버 + 문자열 형식 모두 지원

---

## 참고 자료

- 상위 프로젝트 PRD: `../../LifeRecord/docs/superpowers/specs/2026-03-24-liferecord-design.md`
- WPF 색상 정의: `../../LifeRecord/src/GichanDiary/Resources/Colors.xaml`
- WPF 계산 서비스: `../../LifeRecord/src/GichanDiary/Services/CalculationService.cs`
- HTML UI 목업: `../../LifeRecord/design/ui_mockup_final.html`
