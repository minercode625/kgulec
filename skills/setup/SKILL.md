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
설정 저장 위치: **`~/.claude/kgulec/config.md`** (플러그인 폴더 바깥 → 업데이트가 덮어쓰지 않음).

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
| 기본 언어 | `language` | `ko` 또는 `en` |
| 기본 강의 시간(분) | `lecture_minutes` | 분량 가이드 기준(예: 50/75/150/180=3시간 기본). 이 값은 *기본값*이며, 모드 A가 강의마다 확인해 특강 등은 override한다 |

## 4단계: config 저장

1. `mkdir -p ~/.claude/kgulec` (없으면 생성).
2. 기존 `config.md`가 있으면 `config.md.bak`으로 백업.
3. `config-schema.md`의 형식대로 `~/.claude/kgulec/config.md`를 쓴다:
   - `config_version`은 현재 `schema_version` 값.
   - 2~3단계에서 정한 값 + 사용자가 안 바꾼 기존/기본값을 모두 포함(누락 키 없게).
   - 사용자가 임의로 넣었던 미지의 키는 보존.

## 5단계: (선택) 미리보기

원하면 적용된 테마로 1슬라이드 샘플을 컴파일해 확인한다. 컨텍스트 보호를 위해 **Agent에 위임**한다(임시 폴더에 `shared/beamer-templates.md` 기반 최소 .tex 작성 → PRIMARY THEME 4색을 선택값으로 치환 → `${CLAUDE_PLUGIN_ROOT}/assets/kyonggi_logo.png` 복사 → `pdflatex` → `pdftoppm`로 PNG → Read로 확인 → 결과만 보고). 문제 없으면 정리.

## 6단계: 완료 보고

저장 경로(`~/.claude/kgulec/config.md`), 선택한 테마, 공통 설정값을 한눈에 요약한다. 이후 `/kgulec:new` 생성물에 자동 적용됨을 알린다. 플러그인을 업데이트해도 이 설정은 유지됨을 안내한다.
