# 기찬뷰어 (Gichan Viewer) 프로젝트 작업 규칙

## 프로젝트 개요
- 기찬다이어리(WPF)의 읽기 전용 모바일 동반앱 (PWA)
- 호스팅: GitHub Pages
- 빌드 도구 없음 (Vanilla HTML/CSS/JS)

## 상위 프로젝트 참조
- WPF 소스: `../../LifeRecord/src/GichanDiary/`
- Excel 컬럼 구조: A~Q (17개), 기존 컬럼 변경 금지
- 색상 팔레트: Set 10 로우라이트 (WPF와 동일)
- 계산 로직: CalculationService.cs → calc.js로 포팅

## 기술 스택
- HTML5 + CSS3 + Vanilla JS (ES Modules)
- Chart.js 4.x — 차트 렌더링
- SheetJS (xlsx) — Excel 파싱
- Service Worker — PWA 오프라인
- IndexedDB — 데이터 캐시
- LocalStorage — 설정값

## 화면 구성
- 현황탭: 대시보드 카드 (수유/배변/위생/신체/건강/기타)
- 조회탭: 필터/검색/카드형 리스트/페이지네이션
- 리포트탭: 6종 차트 + 요약 카드 5개
- 설정탭: Excel Import, 페이지당 건수, 아기이름, 테마 전환

## 배포 규칙
- 배포 대상: src/gichan-viewer/ 폴더 전체
- GitHub Pages에서 src/gichan-viewer/를 루트로 설정

## 문서 규칙
- 요구사항: docs/요구사항문서.txt
- PRD: docs/superpowers/specs/2026-03-30-gichan-viewer-design.md
- 작업 기록: docs/작업 메모 기록.txt
- 코드/설계 변경 시 작업 메모 기록 갱신

## Excel 호환성
- 기존 컬럼(A~J) 구조 LifeRecord와 동일
- 확장 컬럼(K~Q) 없는 파일도 지원
- 카테고리 레거시: "위생관리"←"위생", "건강관리"←"통증"

## 배포 정보
- 저장소: https://github.com/keichi-gentle/gichan-viewer (Public)
- main 브랜치: 전체 프로젝트 (docs + src)
- gh-pages 브랜치: PWA 소스만 (subtree push)
- 접속 URL: https://keichi-gentle.github.io/gichan-viewer/
- gh-pages 업데이트 명령: `git subtree push --prefix src/gichan-viewer origin gh-pages`

## 현재 진행 상태
- PWA v1.0 배포 완료 (2026-03-30)
- 스마트폰 정상 동작 확인 (홈 화면 추가 완료)
- 실데이터 테스트 및 UI 피드백 대기 중
