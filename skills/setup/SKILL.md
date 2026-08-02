---
name: setup
description: kgulec 플러그인 초기 설정/테마 변경. 색 프리셋과 교수명·소속·기본 언어·기본 강의시간 같은 공통 설정을 한 번에 정해 사용자 config에 저장한다. /kgulec:setup 로만 실행한다.
disable-model-invocation: true
argument-hint: "[theme|identity|all]"
---

# 모드 D: 셋업 (테마 + 공통 설정) (`/kgulec:setup`)

이 스킬은 **사용자가 명시적으로 `/kgulec:setup`** 했을 때만 실행한다(자동 트리거 안 됨).
강의자료마다 안 바뀌는 **공통 설정**을 모아 사용자 config에 저장한다. **정본 템플릿(`shared/`)은 절대 수정하지 않는다.**

공유 자료 위치: **`${CLAUDE_PLUGIN_ROOT}/shared/`** (이 스킬 기준 `../../shared/`).
설정 저장 위치: **사용자 홈의 `.claude/kgulec/config.md`** (플러그인 폴더 바깥 → 업데이트가 덮어쓰지 않음). macOS/Linux는 `~/.claude/kgulec/`, Windows는 `%USERPROFILE%\.claude\kgulec\`.

> **OS 주의**: 아래 셸 명령 예시는 macOS/Linux(bash) 기준이다. Windows(PowerShell)면 동등 명령을 쓴다 — 폴더 생성 `New-Item -ItemType Directory -Force`, 복사 `Copy-Item`, 삭제 `Remove-Item`. **현재 OS를 확인하고 그에 맞는 명령을 선택해 실행한다.**

## 0단계: 환경 점검 (필수 게이트 — 통과해야 진행)

**`shared/environment-check.md`를 읽어** `pdflatex`(컴파일)와 PDF→PNG 도구(`pdftoppm`/`pdftocairo`/`magick`) 설치 여부를 확인한다. 없으면 그 문서의 절차대로 **OS·패키지 매니저에 맞는 설치 명령을 동의받아** 실행한다(무음 설치 금지).

**이 두 도구는 필수다. 둘 다 설치 확인되기 전에는 1단계로 진행하지 않는다(하드 게이트).**

- 설치 후 **재감지**하여 통과를 확인한 다음에만 1단계로 넘어간다.
- 동의 안 하거나 매니저가 없으면 수동 설치 링크만 안내하고 **setup을 중단**한다. 설치를 마친 뒤 사용자가 다시 `/kgulec:setup`을 실행하도록 안내한다.
- (모드 A/B/C는 비차단 — 도구 없어도 `.tex`는 생성. 하드 게이트는 setup 전용. `environment-check.md`의 「게이트 원칙」 참고.)

## 1단계: 스키마·현재 설정 로드

1. **`shared/config-schema.md`를 읽어** 필드·기본값·스키마 버전을 파악한다.
2. **`~/.claude/kgulec/config.md`를 읽는다**(없으면 신규 작성 모드, 모든 값 기본값에서 시작).
3. **탑업 마이그레이션**: 기존 config의 `config_version < schema_version`이면 `config-schema.md`의 마이그레이션 규칙대로 처리(백업 → 새 필드 채움/필요한 것만 질문).
4. 현재 설정값을 사용자에게 요약해 보여준다(없으면 "설정 없음 — 새로 만듭니다").

인자 `$ARGUMENTS`로 범위를 좁힐 수 있다: `theme`=테마만, `identity`=공통정보만, 그 외/빈값=전체.

## 2단계: 색 테마 선택

1. **`shared/color-presets.md`를 읽어** 프리셋 목록·4색 램프를 가져온다.
2. 사용자에게 프리셋을 제시한다(AskUserQuestion, 각 옵션 `preview`에 base/deep 스와치 hex와 성격 표시). 현재 `theme`를 기본 선택으로.
3. 선택된 프리셋의 4색을 `theme`(이름) + `theme_colors`(base/deep/mid/light RGB)로 확정한다. **이름이 아니라 값을 저장**한다(카탈로그가 바뀌어도 사용자 슬라이드 불변).

## 3단계: 공통 정보 수집

config에 이미 값이 있으면 그 값을 기본으로 제시하고 바꿀지 묻는다(없으면 새로 입력). 수집 항목:

| 항목 | config 키 | 비고 |
|------|-----------|------|
| 교수 이름 | `professor` | 타이틀 `\professor` |
| 소속(학과/학부) | `department` | 타이틀 institute (예: 경기대학교 컴퓨터공학부) |
| **주요 도메인/분야** | `domain` | 후보 토픽·예시 생성 기준(예: 컴퓨터공학·역사·생물학·경영·법학). 자유 입력. 건너뛰면 분야 중립으로 진행 |
| 기본 언어 | `language` | `ko` 또는 `en` |
| 기본 강의 시간(분) | `lecture_minutes` | 분량 가이드 기준(예: 50/75/150/180=3시간 기본). 이 값은 *기본값*이며, 모드 A가 강의마다 확인해 특강 등은 override한다 |

## 4단계: 샘플 미리보기 (필수 — 사용자 확인)

config를 저장하기 **전에**, 확정된 테마로 **샘플 슬라이드 PNG를 만들어 사용자의 작업(프로젝트) 폴더에 저장하고 직접 확인받는다.**

1. **임시 폴더에서 작업한다** — `shared/temp-files-policy.md`대로 작업 폴더에 `./.kgulec-tmp/`를 만들고, 샘플 `.tex`·로고 복사·컴파일 중간물(`.aux`/`.log`/`.pdf`)을 전부 그 안에서 처리한다(시스템 temp 금지 — 백신 오탐 방지).
2. 최소 샘플 .tex를 만든다: `shared/beamer-templates.md` 프리앰블 + 타이틀 1장 + 대표 박스(conceptbox/examplebox/warningbox/mathbox)를 보여주는 1~2장. **PRIMARY THEME 4색을 선택한 `theme_colors`로 치환**하고, `${CLAUDE_PLUGIN_ROOT}/assets/kyonggi_logo.png`를 .tex 옆(`./.kgulec-tmp/`)에 복사한다.
3. 컴파일 → PDF → **PNG 변환** 후 결과만 **현재 작업 폴더**에 `kgulec-theme-sample.png`로 복사한다.
   - 컴파일: `pdflatex`(또는 `latexmk`) — `./.kgulec-tmp/` 안에서.
   - PDF→PNG: 사용 가능한 도구로 변환(`pdftoppm -png -r 150`, 없으면 `pdftocairo -png` 또는 `magick`). **변환 도구가 없으면** `kgulec-theme-sample.pdf`를 대신 작업 폴더로 복사하고 그 사실을 알린다.
   - 컨텍스트 보호: 컴파일/변환 과정은 **Agent에 위임** 가능(결과 파일 경로만 보고받음, **작업 위치 `./.kgulec-tmp/` 명시 지시**).
4. 사용자에게 알린다: "작업 폴더에 `kgulec-theme-sample.png`를 만들었습니다. 열어서 색이 마음에 드는지 확인해주세요." → **확인받는다.**
   - 마음에 안 들면 → **2단계(테마 선택)로 돌아가** 다시 고르고 샘플을 갱신한다(확정될 때까지 반복).
   - 확정되면 **`./.kgulec-tmp/`를 폴더째 삭제**하고(macOS/Linux `rm -rf ./.kgulec-tmp` · Windows `Remove-Item -Recurse -Force .kgulec-tmp`) 5단계로. 작업 폴더의 샘플 PNG는 임시 산출물이니 지워도 된다고 안내한다.
5. 0단계 게이트를 통과했으므로 컴파일·변환 도구는 갖춰져 있다. 만약 이 단계에서 컴파일/변환이 실패하면(도구 손상·패키지 누락 등) 미리보기를 강행하지 말고 원인을 사용자에게 알린 뒤, 필요하면 0단계 점검으로 되돌아간다.

## 5단계: config 저장

1. 홈의 `.claude/kgulec/` 폴더가 없으면 생성한다(macOS/Linux: `mkdir -p ~/.claude/kgulec` · Windows: `New-Item -ItemType Directory -Force $HOME\.claude\kgulec`).
2. 기존 `config.md`가 있으면 `config.md.bak`으로 백업.
3. `config-schema.md`의 형식대로 `config.md`를 쓴다:
   - `config_version`은 현재 `schema_version` 값.
   - 2~3단계에서 정한 값 + 사용자가 안 바꾼 기존/기본값을 모두 포함(누락 키 없게).
   - 사용자가 임의로 넣었던 미지의 키는 보존.

## 6단계: 완료 보고

저장 경로(`~/.claude/kgulec/config.md`), 선택한 테마, 공통 설정값을 한눈에 요약한다. 이후 `/kgulec:new` 생성물에 자동 적용됨을 알린다. 플러그인을 업데이트해도 이 설정은 유지됨을 안내한다.
