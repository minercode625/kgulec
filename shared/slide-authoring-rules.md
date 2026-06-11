# 슬라이드 작성 공통 규칙 (모드 A·B)

.tex를 작성·수정하는 모드 A·모드 B에서 공통으로 적용하는 디자인·분량·레이아웃 규칙이다.

## 디자인 시스템

시각적 핵심은 **tcolorbox 기반 색상 코딩 시스템**이다:

| 박스 스타일 | 색상 | 용도 |
|------------|------|------|
| `conceptbox` | 파란색 | 핵심 개념, 정의, 정리, 메인 아이디어 |
| `examplebox` | 초록색 | 예제, 좋은 사례, 결과 확인 |
| `warningbox` | 주황색 | 주의사항, 흔한 실수, 질문 |
| `infobox` | 회색 | 보충 정보, 직관, 팁 |
| `mathbox` | 연한 파랑 | 수식 강조 블록 |
| `taskcard` | 사용자 지정 색 | 아이콘 + 짧은 텍스트 카드 |

**박스 타이틀 FontAwesome 아이콘 예시:**
- `\faIcon{lightbulb}` 직관/팁, `\faIcon{check-circle}` 핵심요점, `\faIcon{exclamation-triangle}` 주의, `\faIcon{search}` 탐색, `\faIcon{balance-scale}` 비교, `\faIcon{table}` 데이터, `\faIcon{robot}` AI, `\faIcon{question-circle}` 질문, `\faIcon{book-open}` 개요, `\faIcon{route}` 핵심 아이디어

---

## 슬라이드 분량 가이드

| 강의 시간 | 권장 슬라이드 수 |
|-----------|----------------|
| 15분 | 10~15장 |
| 50분 (1교시) | 25~35장 |
| 75분 (1.5교시) | 35~50장 |
| 100분 (2교시) | 50~70장 |
| 150분 (3교시) | 70~95장 |
| 180분 (3시간·기본) | 85~115장 |
| 지정 없음 | 기본 3시간으로 가정 (모드 A가 이번 강의 시간 확인) |

---

## 오버플로우 방지

콘텐츠 높이 한도는 **7.0cm** (4:3 Beamer, frametitle + footline 제외). Beamer는 자동 줄임이 없으므로 반드시 지켜야 한다.

### 높이 예산표

| 요소 | 대략적 높이 |
|------|------------|
| 제목 텍스트 + vspace | 0.5~0.6cm |
| tcolorbox (2줄, `\scriptsize`) | 1.0~1.3cm |
| tcolorbox (4줄, `\scriptsize`) | 1.5~1.8cm |
| columns (2열, 각 tcolorbox) | 1.5~2.5cm |
| TikZ (scale=0.7~0.8, 노드 4~6개) | 1.5~2.8cm |
| algorithm2e (6~8줄) | 2.5~3.5cm |
| tabular (헤더 + 3행) | 1.5~2.0cm |

### 자가 점검

프레임 작성 후 높이를 합산한다:
- tcolorbox: 줄 수 × 0.35cm + 타이틀 0.4cm
- TikZ: scale × 3cm (대략)
- **6.5cm 이하 = 안전, 7.0cm 초과 = 반드시 수정** (슬라이드 분리 또는 scale/폰트 축소)

### 높이 절약 기법

- TikZ를 tcolorbox 안에 넣으면 여백 절약
- `\vspace{-0.1cm}`으로 frametitle 아래 여백 축소
- tcolorbox에 `top=2pt, bottom=2pt` 추가
- `\scriptsize` 기본 사용 (`\footnotesize` 대비 ~15% 절약)
- TikZ scale 0.7~0.75 사용
- `\setlength{\itemsep}{1pt}`로 항목 간격 최소화

---

## 주의사항

- .tex 파일은 반드시 컴파일 가능해야 한다.
- 프리앰블은 템플릿을 그대로 사용한다. 절대 간소화하지 않는다.
- `\rowcolor`를 사용하는 경우 `\documentclass[xcolor={svgnames,table}]{beamer}`로 변경해야 한다 (기본 `xcolor=svgnames`에서는 `\rowcolor`가 작동하지 않음).
- `\pause` 사용 금지. columns + tcolorbox로 표현한다.
- `listings` 사용 금지. `algorithm2e` 또는 `verbatim`을 사용한다.
- `metropolis` 테마 사용 금지. Madrid + 커스텀 색상을 사용한다.
- `\vspace`로 적절히 여백을 준다 (보통 0.08cm ~ 0.20cm).
- `\scriptsize`, `\footnotesize`, `\small` 등 폰트 크기를 적절히 사용한다.
- **모든 프레임은 오버플로우 방지 섹션의 높이 예산을 준수한다.**
