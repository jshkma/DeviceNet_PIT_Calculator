# CLAUDE.md

이 파일은 Claude Code (claude.ai/code)가 이 저장소에서 작업할 때 참고하는 안내 문서입니다.

## 애플리케이션 실행 방법

`DeviceNet_PIT_Calculator.html` 파일을 최신 웹 브라우저에서 직접 열면 됩니다. 빌드 과정, 패키지 매니저, 서버가 필요하지 않습니다.

## 프로젝트 구조

이 프로젝트는 **단일 파일 웹 애플리케이션**으로, HTML·CSS·JavaScript가 모두 `DeviceNet_PIT_Calculator.html`(약 578줄)에 포함되어 있습니다. 외부 의존성이나 별도의 소스 파일은 없습니다.

- `DeviceNet_PIT_Calculator.html` — 전체 애플리케이션
- `DeviceNet_PIT_Calculator_Manual.docx` — 사용자 매뉴얼 (한국어)

## 아키텍처

파일은 세 개의 순차적 섹션으로 구성됩니다:

1. **CSS (7–211번 줄):** CSS 사용자 정의 속성(변수)을 활용한 인라인 스타일. 색상 테마 및 반응형 그리드 레이아웃, 인쇄용 미디어 쿼리 포함.
2. **HTML (214–393번 줄):** 네 개의 카드 — 네트워크 설정, 노드 관리, 결과, 도움말. 노드 행은 `#nodeTableBody`에 동적으로 생성됨.
3. **JavaScript (400–576번 줄):** 모든 로직이 단일 `<script>` 블록 안에 존재. 모듈·클래스 없이 일반 함수와 두 개의 전역 변수(`nodeId`, `nodes` 배열)만 사용.

## 핵심 알고리즘

**프레임 시간** (`frameTime(bytes, baud)`):
- CAN 프레임 비트 수 = 47 + bytes × 8
- 비트 스터핑 계수 1.2× 적용
- 밀리초 변환: `(stuffed_bits / (baud_kbps × 1000)) × 1000`

**노드 전송 시간** (`nodeTransferTime(inB, outB, baud)`):
- 요청(마스터→슬레이브)과 응답(슬레이브→마스터) 프레임 시간의 합산
- 페이로드가 8바이트 초과 시 여러 8바이트 프레임으로 분할 (`fragCount(b)`)

**메인 계산** (`calc()`):
- 모든 노드 전송 시간 합산 → 총 폴 점유 시간
- 안전 여유율 곱하기 → 권장 PIT 값
- EPR 타임아웃 상한 = EPR × 4 ms
- 판정: PIT ≤ EPR 상한이면 초록, 근접하면 노랑, 초과 시 빨강
- 버스 부하율 = 총 폴 시간 / PIT × 100%

## 도메인 용어

- **DeviceNet / CAN bus** — 산업용 필드버스 프로토콜
- **CIF50-DNM** — Hilscher 마스터 인터페이스 카드
- **EPR** (Expected Packet Rate) — SyConDN에서 설정; 타임아웃 = EPR × 4 ms
- **PIT** (Poll Interval Time) — 이 도구가 최적값을 계산하는 대상
- 인터페이스 언어는 한국어이며, 대상 사용자는 TES CO., LTD. 엔지니어
