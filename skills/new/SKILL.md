---
name: new
description: 새 LaTeX Beamer 대학 강의자료(.tex 슬라이드)를 만들 때 사용. 주제만 주거나, 강의 노트·원고를 주거나, 기존 파일을 업로드해도 새 슬라이드를 생성한다. "강의자료 만들어줘", "슬라이드 만들어줘", "beamer 강의자료", "수업 자료 생성", "발표자료 tex" 등의 요청. CS/공학 대학 강의 슬라이드 신규 작성이면 LaTeX/Beamer 언급이 없어도 트리거한다.
argument-hint: "[과목/주제]"
---

# 모드 A: 강의자료 새로 만들기 (`/kgulec:new`)

공유 자료 위치: **`${CLAUDE_PLUGIN_ROOT}/shared/`** (이 스킬 기준 `../../shared/`). 아래에서 `shared/`로 표기.
.tex 작성 규칙은 **`shared/slide-authoring-rules.md`**(디자인·분량·오버플로우·주의)를 반드시 함께 읽는다.

## 0단계: 사용자 설정(config) 로드 + 마이그레이션 (가장 먼저)

1. **`shared/config-schema.md`를 읽어** 스키마와 기본값을 파악한다.
2. **`~/.claude/kgulec/config.md`를 읽는다.**
   - 없으면: `/kgulec:setup` 실행을 권한다("공통 설정이 없습니다. /kgulec:setup 을 먼저 하면 매번 안 물어봅니다"). 사용자가 그냥 진행하면 모든 값을 스키마 기본값으로 가정한다.
   - 있으면: **defaults ⊕ config 병합**(누락 키는 기본값). 절대 크래시 금지.
3. **탑업 마이그레이션**: `config.config_version < schema_version`이면 `shared/config-schema.md`의 "마이그레이션 규칙"대로 처리(백업 → 새 필드 채움/필요한 것만 질문 → 버전 범프 후 저장).

config가 제공하는 값(`professor`, `department`, `language`, `lecture_minutes`, `theme`/`theme_colors`)은 **1단계에서 재질문하지 않는다.**

## 1단계: 입력 수집 및 작업 폴더 준비

이 플러그인은 경기대 교수들이 공용으로 사용한다. **`weekN/` 같은 주차별 폴더 구조를 강제하지 않으며**, 사용자가 `./figs`·`./ref` 폴더의 존재나 용도를 모를 수 있다고 가정한다.

### 1-1. 작업 폴더 준비 + 자료 폴더 자동 생성

1. **출력(컴파일) 디렉토리 결정**: 기본은 현재 작업 디렉토리. 주차별 폴더를 강제하지 않는다.
2. **자료 폴더 자동 생성**: 출력 디렉토리에 `./figs`, `./ref`가 없으면 만든다(`mkdir -p figs ref`).
3. **사용자에게 먼저 물어본다**: "넣을 **그림**이 있으면 `./figs/`에, **참고자료**(PDF·슬라이드·강의노트·교재 발췌)가 있으면 `./ref/`에 넣어주세요. 있으신가요?"
   - 있으면 사용자가 파일을 넣을 때까지 기다린 뒤 스캔. 없으면 웹 검색 위주로 진행.

### 1-2. 주변 파일 스캔

질문 전 출력 디렉토리를 확인해 추론 가능한 것은 추론하고 **확인만** 받는다.
- `./ref` → 1차 출처 자료(주제 단서) · `./figs` → 그림 목록
- 기존 `main.tex`/강의계획서/`.md` 노트 → 과목명·주제·주차
- 폴더명이 `weekN` 형태면 주차(`\week`) 추론(best-effort, 아니면 1-3에서 질문)
- `assets/kyonggi_logo.png` → 로고 유무

### 1-3. 확인되지 않은 항목만 질문 (config·스캔으로 확정된 건 생략)

| 항목 | 매크로 | 비고 |
|------|--------|------|
| **과목명** | `\course` | 타이틀 메인. 강의별로 다를 수 있어 config에 없음 → 보통 질문 |
| **대주제 / 세부 주제** | `\topic` | 핵심 내용. **없으면 예시 제시**(아래) |
| **다음 주차 대주제·세부** | — | 마지막 "다음 주차 예고" 슬라이드용 |
| **주차 번호** | `\week` | 폴더명에서 추론되면 확인만 |
| **강의 날짜** | `\naljja` | 미지정 시 오늘 날짜 제안 후 확인 |
| (교수명·소속·언어·강의시간) | — | **config에 있으면 질문 생략.** 없을 때만 물어보고, 자주 쓰면 /kgulec:setup 권유 |

**대주제·세부 주제가 없으면** 임의로 지어내지 말고 후보를 예시로 제시해 고르게 한다.
예: "딥러닝개론 N주차라면 — ① CNN 기초 ② RNN/LSTM ③ Attention/Transformer 중 무엇으로 할까요?"

### 1-4. 함께 오는 입력 형태

| 입력 유형 | 예시 | 처리 |
|-----------|------|------|
| **주제만** | "Transformer 강의자료 만들어줘" | 핵심 개념 구조화하여 생성 |
| **강의 노트/원고** | 텍스트 설명·개조식 메모 | 슬라이드 단위로 분할·재구성 |
| **파일 업로드** | .txt/.md/.pdf/.tex | 내용 읽고 Beamer로 변환 |

## 2단계: 자료 조사

충분히 인터넷 검색하여 (config의 `lecture_minutes` 기준) 강의 분량에 맞는 내용을 포괄 조사한다. 모든 내용은 사실 기반(확실치 않으면 추측 금지, 검증 후 포함).

**`./ref` 우선(필수)**: 있으면 그 자료(PDF는 Read로 직접)를 **1차 출처**로 삼아 범위·용어·표기·예시를 일관되게. 웹 검색은 보완용. 현재 주제와 직접 관련된 내용만 사용. 큰 PDF는 Agent에 요약 위임. `./ref` 내용도 오래/부정확하면 웹으로 교차검증.

## 3단계: Outline 제시 및 승인

3시간(또는 `lecture_minutes`) 분량 Outline(섹션 구성·핵심 내용·슬라이드 수 예상)을 제시하고 **사용자 승인** 후 진행.

## 4단계: 언어 결정

config `language`를 기본으로 한다. 사용자가 명시 지정하면 그에 따른다. 한글은 `kotex`로 지원(프리앰블 포함).

## 5단계: 슬라이드 구조 설계

전형 흐름: Title → 목차(`\tableofcontents`) → Section divider(`\sectiondividerframe`) → Content(tcolorbox) → Summary/Next Class. 슬라이드당 과밀 금지. tcolorbox + columns로 구조화.

## 6단계: .tex 파일 생성 + config 오버레이

1. **`shared/beamer-templates.md`를 먼저 읽고** 표준 프리앰블을 그대로 사용한다. 간소화·임의 변경 금지.
2. **config 오버레이** (생성하는 main.tex에 적용):
   - `\professor` ← config `professor`, `\department` ← config `department`
   - `\course`/`\week`/`\topic`/`\naljja` ← 1단계 입력
   - **PRIMARY THEME 4색** ← config `theme_colors`(base→`mqblue`, deep→`mqdeepblue`, mid→`mqgrayblue`, light→`mqlightblue`)로 `% === PRIMARY THEME ===`~`% === END PRIMARY THEME ===` 4줄 치환. 의미색·`\colorlet`은 그대로.
3. **로고 배치(필수)**: `\titlegraphic`은 `main.tex`와 같은 폴더의 `kyonggi_logo.png`를 찾는다. `${CLAUDE_PLUGIN_ROOT}/assets/kyonggi_logo.png`를 출력 디렉토리로 복사한다(`cp "${CLAUDE_PLUGIN_ROOT}/assets/kyonggi_logo.png" ./`). 없어도 `\IfFileExists`라 컴파일 안 깨짐.
4. **`shared/gotcha/`의 .md를 모두 읽고** 동일 실수를 반복하지 않는다.

핵심 원칙:
- **컴파일 가능한 완전한 .tex** 생성.
- tcolorbox 스타일(conceptbox/examplebox/warningbox/infobox/mathbox/taskcard) + FontAwesome5 아이콘 적극 활용.
- `\pause` 금지(columns+tcolorbox로). `listings` 금지(`algorithm2e`/`verbatim`).
- **기존 `./figs/` 우선**: 현재 강의와 직접 관련된 이미지만 `\includegraphics[width=...]{figs/파일명.png}`로 삽입. 한 이미지를 무관한 여러 곳에 중복 금지.
- 적합한 그림이 없으면 TODO 주석 + 공간 확보:
  ```latex
  % TODO [IMAGE]: 검색 키워드 — 이미지 설명
  % 교체: ./figs/에 저장 후 아래 주석 해제
  % \includegraphics[width=0.8\textwidth]{figs/파일명.png}
  ```

## 7단계: 품질 검사 (자동)

컴파일(`pdflatex`/`latexmk`) 후 PDF에 대해: ① 내용 정확성 ② 레이아웃 오버플로 ③ figure 렌더링. 문제 슬라이드는 자동 수정·재컴파일.

**오버플로 검사는 Agent에 위임**: 메인에서 .tex 수정·컴파일 완료 → Agent 호출(PDF 경로 + 페이지 범위 + `pdftoppm`으로 PNG 변환 후 Read 확인 + 오버플로 페이지·증상만 보고) → 결과 받아 해당 프레임 수정·재컴파일. (메인 컨텍스트에 이미지 안 쌓이게.)

## 8단계: Gotcha 기록 (필수 — 건너뛰지 않음)

컴파일/품질검사에서 발견·해결한 문제 중 `shared/gotcha/`에 없는 **새 에러**만 `.md`로 기록(증상·원인·해결). 기존 파일 먼저 확인. 새 에러 없어도 이 단계는 의식적으로 확인하고 넘어간다.
