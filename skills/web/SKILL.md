---
name: web
description: 이미 만든 LaTeX Beamer 강의자료(main.tex)를 움직이는 HTML 프레젠테이션(slides.html)으로 변환할 때 사용. 애니메이션·순차 등장·섹션 전환 효과가 있는 단일 HTML 파일을 만든다. "움직이는 ppt", "HTML 슬라이드", "웹 프레젠테이션", "애니메이션 슬라이드", "인터랙티브 발표자료", "reveal로 바꿔줘", "브라우저에서 발표" 등의 요청이면 트리거한다. 기존 slides.html의 수정 요청("웹 슬라이드 고쳐줘")도 이 모드가 담당한다.
argument-hint: "[변환할 main.tex 경로 (생략 시 현재 폴더)]"
---

# 모드 E: 움직이는 HTML 프레젠테이션 (`/kgulec:web`)

공유 자료 위치: **`${CLAUDE_PLUGIN_ROOT}/shared/`** (이 스킬 기준 `../../shared/`). 아래에서 `shared/`로 표기.
템플릿·CSS·빌드 스크립트·변환 매핑은 **`shared/html-slides-template.md`**가 단일 진실 공급원이다 — **반드시 먼저 읽고 그대로 사용한다**(간소화·임의 변경 금지).

> **OS 주의**: 셸 명령은 macOS/Linux(bash) 기준 예시이며 Windows(PowerShell)면 동등 명령을 쓴다(템플릿 문서에 쌍으로 수록). Python 실행은 macOS에서 `python3`, Windows에서 `py`(또는 `python`).

## 무엇을 만드나

기존 `main.tex`(Beamer)를 **내용·순서 1:1 유지**한 채 애니메이션 있는 **단일 자립 `slides.html`**로 변환한다(reveal.js 인라인 + 이미지 base64, 네트워크 불필요). `main.tex`/`main.pdf`는 절대 건드리지 않는다 — 별도 산출물이다.

## 1단계: 입력 확인

1. 대상 `main.tex`를 찾는다(기본: 현재 폴더). **없으면 변환 불가** — "변환할 main.tex가 없습니다. `/kgulec:new`로 먼저 강의자료를 만드세요"라고 안내하고 종료한다. PDF만으로는 변환하지 않는다.
2. `main.tex` 전체를 읽고 구조를 파악한다: 슬라이드 수, 섹션(디바이더) 수, 사용된 이미지(`figs/`, 로고), 표·verbatim·TikZ 유무, **PRIMARY THEME 4색 값**.
3. 이미 `web_src/`·`slides.html`이 있으면 **수정 모드**다: 파트 파일을 고치고 재빌드한다(새로 만들지 않음).

## 2단계: 옵션 확인 (AskUserQuestion)

기본값으로 바로 진행해도 되는 항목이지만, 처음 변환하는 폴더면 한 번에 묶어 확인한다:

- **애니메이션 수준**: ① 적당히 생동감(기본 — 순차 등장 + 디바이더 애니) ② 화려하게 ③ 최소(전환만)
- **변환 범위**: ① 전체 1:1(기본) ② 일부 섹션만

수정 모드거나 사용자가 이미 지정했으면 질문 생략.

## 3단계: 변환·생성

`shared/html-slides-template.md`의 규칙을 그대로 따른다. 요점:

1. `web_src/` 생성 → `${CLAUDE_PLUGIN_ROOT}/assets/web/`의 `reveal.js`·`reveal.css`·`reset.css` 복사 → `build.py` 작성(템플릿 문서의 것 그대로).
2. **테마 오버레이**: main.tex의 PRIMARY THEME 4색을 CSS `:root` 변수(`--mqblue`/`--mqdeep`/`--mqgray`/`--mqlight`)로 치환. 의미색은 고정 — config를 다시 읽을 필요 없음(main.tex에 이미 반영돼 있음).
3. 슬라이드를 part1~3.html에 나눠 작성(템플릿의 「Beamer → HTML 변환 매핑」 표대로). 타이틀 메타(`\course`/`\topic`/`\professor` 등)는 타이틀 슬라이드와 하단바(footbar)에 주입.
4. **⛔ 꽉 찬 레이아웃 규칙(하드 룰) 준수** — 템플릿 문서의 해당 절이 최우선이다: 프레임 풀높이(676px) + `space-evenly` 배분, 폰트 기준값(본문 22px 등) 이하로 줄여 여백 만들기 금지, 콘텐츠 적은 슬라이드는 폰트·이미지를 **키워서** 채움, 위아래 죽은 여백 금지. 오버플로 시에만 해당 슬라이드 한정 소폭 축소.
5. 애니메이션은 템플릿의 「애니메이션 규칙」(2단계에서 고른 수준 반영).
6. `build.py` 실행 → `slides.html` 생성 확인(이미지 임베드 수 포함).

## 4단계: 품질 검사

템플릿 문서의 「품질 검사」 절대로: 로컬 서버(`python3 -m http.server`) + 가능하면 Playwright로 **타이틀·디바이더·밀도 최고/최저·마지막** 슬라이드를 fragment 전부 표시 상태(`Reveal.slide(h,0,99)`)로 스크린샷 확인한다 — **여백 과다와 넘침 둘 다 결함**이다. 문제 슬라이드는 파트 파일 수정 → 재빌드 → 재확인. 시각 확인은 Agent에 위임 가능(메인 컨텍스트에 이미지 안 쌓이게).

**임시 파일 정책(필수 — `shared/temp-files-policy.md`)**: 스크린샷 등 검사용 임시 파일은 시스템 temp가 아니라 **작업 폴더의 `./.kgulec-tmp/`에 저장**한다(Playwright 저장 경로를 이 폴더로 지정, Agent 위임 시에도 경로 명시). 검사 통과 후 **폴더째 삭제**한다. 렌더 확인이 불가한 환경이면 사용자에게 확인을 요청한다. 검사 후 서버는 반드시 종료.

## 5단계: 마무리 안내 + Gotcha

1. 사용자에게 알린다: 산출물 위치, 더블클릭 실행, 방향키/ESC/전체화면 조작, `?print-pdf` PDF 백업, 이후 수정은 `/kgulec:web`으로(파트 수정 → 재빌드).
2. **Gotcha 기록**: 변환·렌더 검증에서 발견·해결한 **새 문제**(번들 `shared/gotcha/` + 사용자 `~/.claude/kgulec/gotcha/` 어디에도 없는 것)는 모드 A 8단계와 동일 규칙으로 **사용자 폴더**에 기록한다(파일명 `web-` 접두사 권장, 용량 다이어트·중복 체크·병합 규칙 동일).
