---
name: new
description: 새 LaTeX Beamer 대학 강의자료(.tex 슬라이드)를 만들 때 사용. 주제만 주거나, 강의 노트·원고를 주거나, 기존 파일을 업로드해도 새 슬라이드를 생성한다. "강의자료 만들어줘", "슬라이드 만들어줘", "beamer 강의자료", "수업 자료 생성", "발표자료 tex" 등의 요청. CS/공학 대학 강의 슬라이드 신규 작성이면 LaTeX/Beamer 언급이 없어도 트리거한다.
argument-hint: "[과목/주제]"
---

# 모드 A: 강의자료 새로 만들기 (`/kgulec:new`)

공유 자료 위치: **`${CLAUDE_PLUGIN_ROOT}/shared/`** (이 스킬 기준 `../../shared/`). 아래에서 `shared/`로 표기.
.tex 작성 규칙은 **`shared/slide-authoring-rules.md`**(디자인·분량·오버플로우·주의)를 반드시 함께 읽는다.

> **OS 주의**: 아래 셸 명령 예시는 macOS/Linux(bash) 기준이다. Windows(PowerShell)면 동등 명령을 쓴다 — 폴더 생성 `New-Item -ItemType Directory -Force`, 복사 `Copy-Item`. config 경로는 macOS/Linux `~/.claude/kgulec/`, Windows `%USERPROFILE%\.claude\kgulec\`. **현재 OS를 확인하고 그에 맞는 명령을 선택해 실행한다.**

> **질문 방식**: 사용자에게 **결정·승인·후보 선택**을 요청할 때는 자유 텍스트로 묻지 말고 **`AskUserQuestion`으로 선택지**를 제시한다(Outline 승인, 강의 시간 확인, 대주제 후보 고르기, figs/ref 유무 등). 자유 입력이 본질인 값(과목명·주제 문구 직접 입력 등)만 일반 질문으로 받는다.

## 0단계: 사용자 설정(config) 로드 + 마이그레이션 (가장 먼저)

1. **`shared/config-schema.md`를 읽어** 스키마와 기본값을 파악한다.
2. **`~/.claude/kgulec/config.md`를 읽는다.**
   - 없으면: `/kgulec:setup` 실행을 권한다("공통 설정이 없습니다. /kgulec:setup 을 먼저 하면 매번 안 물어봅니다"). 사용자가 그냥 진행하면 모든 값을 스키마 기본값으로 가정한다.
   - 있으면: **defaults ⊕ config 병합**(누락 키는 기본값). 절대 크래시 금지.
3. **탑업 마이그레이션**: `config.config_version < schema_version`이면 `shared/config-schema.md`의 "마이그레이션 규칙"대로 처리(백업 → 새 필드 채움/필요한 것만 질문 → 버전 범프 후 저장).

config가 제공하는 값(`professor`, `department`, `language`, `theme`/`theme_colors`)은 **1단계에서 재질문하지 않는다.**

단 **강의 시간(`lecture_minutes`)은 예외**다. config 값은 *기본값*일 뿐, 이번 강의에는 다를 수 있다(예: 평소 3시간이지만 특강은 1시간). 따라서 1단계에서 **이번 강의 시간을 확인**한다(아래 1-3 참조).

## 1단계: 입력 수집 및 작업 폴더 준비

이 플러그인은 경기대 교수들이 공용으로 사용한다. **`weekN/` 같은 주차별 폴더 구조를 강제하지 않으며**, 사용자가 `./figs`·`./ref` 폴더의 존재나 용도를 모를 수 있다고 가정한다.

### 1-1. 작업 폴더 준비 + 자료 요청 (가장 먼저)

1. **출력(컴파일) 디렉토리 결정**: 기본은 현재 작업 디렉토리. 주차별 폴더를 강제하지 않는다.
2. **자료 폴더 먼저 생성**: 출력 디렉토리에 `figs`, `ref`가 **없으면 먼저 만든다**(macOS/Linux: `mkdir -p figs ref` · Windows: `New-Item -ItemType Directory -Force figs, ref`).
3. **자료 배치를 요청한다** — 자유 텍스트로 묻지 말고 `AskUserQuestion` 선택지로:
   - 안내: "`./figs/`에는 강의에 쓸 **이미지**를, `./ref/`에는 강의자료 생성에 참고할 **문헌 자료(PDF)**를 넣어주세요."
   - 선택지 예: ① **자료를 넣었어요(또는 지금 넣을게요)** · ② **자료 없음 — 웹 검색만으로 진행**
4. **분기**:
   - ① → 사용자가 파일을 다 넣을 때까지 기다린 뒤 **1-2에서 두 폴더를 다시 확인(스캔)**한다.
   - ② → 두 폴더는 빈 채로 두고 웹 검색 위주로 진행(폴더 스캔 생략 가능). **반드시 "없음" 응답도 받을 수 있어야 한다.**

### 1-2. 자료 폴더·주변 파일 재확인 (스캔)

1-1에서 ①을 골랐으면 `./figs`·`./ref`를 **다시 확인**하여 실제 들어온 파일을 파악한다. 그 외 주변 파일도 스캔해 추론 가능한 값은 추론하고 **확인만** 받는다.
- `./ref` → 1차 출처 자료(주제 단서, PDF는 Read로 직접) · `./figs` → 사용 가능한 그림 목록
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
| **강의 시간** | — | **이번 강의 길이를 항상 확인한다.** config `lecture_minutes`를 기본값으로 제시하고 "이번 강의도 N분인가요? (특강 등 다르면 알려주세요)"로 확인 → 다르면 그 값으로 override(특강 1시간 등). config에 없으면 기본 3시간(180분)을 제안하며 묻는다. 확정값은 슬라이드 수 기준(`shared/slide-authoring-rules.md` 분량 가이드)이며, config는 덮어쓰지 않는다(이번 강의 한정). |
| (교수명·소속·언어) | — | **config에 있으면 질문 생략.** 없을 때만 물어보고, 자주 쓰면 /kgulec:setup 권유 |

**대주제·세부 주제가 없으면** 임의로 지어내지 말고 후보를 예시로 제시해 고르게 한다.
예: "딥러닝개론 N주차라면 — ① CNN 기초 ② RNN/LSTM ③ Attention/Transformer 중 무엇으로 할까요?"

### 1-4. 함께 오는 입력 형태

| 입력 유형 | 예시 | 처리 |
|-----------|------|------|
| **주제만** | "Transformer 강의자료 만들어줘" | 핵심 개념 구조화하여 생성 |
| **강의 노트/원고** | 텍스트 설명·개조식 메모 | 슬라이드 단위로 분할·재구성 |
| **파일 업로드** | .txt/.md/.pdf/.tex | 내용 읽고 Beamer로 변환 |

## 2단계: 자료 조사

충분히 인터넷 검색하여 (1단계에서 확정한 이번 강의 시간 기준) 강의 분량에 맞는 내용을 포괄 조사한다. 모든 내용은 사실 기반(확실치 않으면 추측 금지, 검증 후 포함).

**`./ref` 우선(필수)**: 있으면 그 자료(PDF는 Read로 직접)를 **1차 출처**로 삼아 범위·용어·표기·예시를 일관되게. 웹 검색은 보완용. 현재 주제와 직접 관련된 내용만 사용. 큰 PDF는 Agent에 요약 위임. `./ref` 내용도 오래/부정확하면 웹으로 교차검증.

## 3단계: Outline 제시 및 승인

확정한 이번 강의 시간(기본 3시간) 분량 Outline(섹션 구성·핵심 내용·슬라이드 수 예상)을 제시한다. Outline 본문은 텍스트로 보여주되, **승인은 자유 텍스트로 묻지 말고 `AskUserQuestion`으로 선택지를 준다.** 예:

- **이대로 진행** (권장)
- **스코프 조정** — 특정 섹션 추가/제외/비중 변경 (선택 시 무엇을 바꿀지 이어서 질문)
- **분량·강의 시간 변경** — 슬라이드 수/시간 재설정

(사용자는 'Other'로 자유 입력도 가능.) 선택에 따라 Outline을 수정해 다시 제시하고, **이대로 진행**이 선택돼야 다음 단계로 넘어간다.

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
3. **로고 배치(필수)**: `\titlegraphic`은 `main.tex`와 같은 폴더의 `kyonggi_logo.png`를 찾는다. `${CLAUDE_PLUGIN_ROOT}/assets/kyonggi_logo.png`를 출력 디렉토리로 복사한다(macOS/Linux: `cp "${CLAUDE_PLUGIN_ROOT}/assets/kyonggi_logo.png" ./` · Windows: `Copy-Item "$env:CLAUDE_PLUGIN_ROOT\assets\kyonggi_logo.png" .`). 없어도 `\IfFileExists`라 컴파일 안 깨짐.
4. **번들 `shared/gotcha/` + 사용자 `~/.claude/kgulec/gotcha/`(있으면)의 .md를 모두 읽고** 동일 실수를 반복하지 않는다.

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

**컴파일 전 환경 점검**: `pdflatex`·PDF→PNG 도구가 있는지 확인한다(`shared/environment-check.md`). 없으면 그 문서대로 동의받아 설치 제안, 끝내 없으면 `.tex`까지만 생성하고 "설치 후 `pdflatex main.tex`로 컴파일하세요" 안내 후 종료(작업은 막지 않음).

컴파일(`pdflatex`/`latexmk`) 후 PDF에 대해: ① 내용 정확성 ② 레이아웃 오버플로 ③ figure 렌더링. 문제 슬라이드는 자동 수정·재컴파일.

**오버플로 검사는 Agent에 위임**: 메인에서 .tex 수정·컴파일 완료 → Agent 호출(PDF 경로 + 페이지 범위 + PDF를 PNG로 변환[`pdftoppm`/`pdftocairo`/`magick` 중 가능한 것] 후 Read 확인 + 오버플로 페이지·증상만 보고) → 결과 받아 해당 프레임 수정·재컴파일. (메인 컨텍스트에 이미지 안 쌓이게.)

**섹션 표지(divider) 재확인 (필수 — 자주 엇나감)**: `\sectiondividerframe`은 `[plain]` + 인라인 `remember picture,overlay`로 배경을 그려 **특히 뒤쪽 섹션 표지에서 배경이 어긋난다**(상단 회색 머리띠, 파란 밴드가 위로 밀림, 하단 흰색 잘림). 마지막에 **각 섹션의 첫 표지 페이지를 하나도 빠짐없이 따로** PNG로 확인한다.
- 점검 항목: ① 전체 배경(`mqdeepblue`)이 페이지 끝까지 채워짐 ② 파란 밴드(`mqlightblue`) 위치 정상 ③ 상단 회색 머리띠 없음 ④ 하단 흰 여백 잘림 없음.
- overlay는 `current page` 앵커가 cross-ref라 **최소 2~3회 컴파일**해야 위치가 수렴한다. 한 번만 컴파일하면 엇나간 채로 남으니, 표지 확인 전 반드시 다회 컴파일.
- 다회 컴파일에도 일부 표지가 계속 어긋나면 → `shared/gotcha/tikz-pitfalls.md`의 「overlay 배경」 해법대로 그 표지의 overlay를 `\setbeamertemplate{background canvas}`로 옮긴다.

## 8단계: Gotcha 기록 (필수 — 건너뛰지 않음)

컴파일/품질검사에서 발견·해결한 문제 중 **번들·사용자 gotcha 어디에도 없는 새 에러**만 `.md`로 기록(증상·원인·해결).

- **쓰는 위치 = 사용자 폴더** `~/.claude/kgulec/gotcha/`(Windows `%USERPROFILE%\.claude\kgulec\gotcha\`). 없으면 만든다(macOS/Linux `mkdir -p ~/.claude/kgulec/gotcha` · Windows `New-Item -ItemType Directory -Force $HOME\.claude\kgulec\gotcha`).
- **번들 `shared/gotcha/`(읽기전용)에는 쓰지 않는다** — 플러그인 업데이트 시 사라지고 사용자 git도 아니다(config와 동일 원칙).
- 중복 금지: 기존(번들 + 사용자) 먼저 확인.
- 새 에러 없어도 이 단계는 의식적으로 확인하고 넘어간다.
