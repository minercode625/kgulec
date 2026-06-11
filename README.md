# kgulec — KGU Beamer Lecture

경기대학교 대학 강의(CS/공학)를 위한 **LaTeX Beamer 슬라이드 생성·수정·실습코드·테마 셋업 Claude Code 플러그인**.

주제만 줘도, 강의 노트·원고를 줘도, 기존 `.tex`를 줘도 대응한다. 수식(amsmath), 알고리즘(algorithm2e), TikZ 다이어그램, tcolorbox 색상 코딩 디자인을 활용하며 한글·영어를 모두 지원한다. 경기대 내 공용·오픈소스 사용을 목적으로 한다.

## 명령어

| 명령 | 모드 | 하는 일 | 트리거 |
|------|------|---------|--------|
| `/kgulec:new` | A 생성 | 자료 조사 → Outline 승인 → 슬라이드 생성 → 컴파일·품질검사 | 명시 + 자연어 자동 |
| `/kgulec:edit` | B 수정 | 기존 `main.tex` 수정·삭제·순서변경·교체·보강 | 명시 + 자연어 자동 |
| `/kgulec:code` | C 실습 | 강의 내용 기반 실습 프로그램(Python/HTML) | 명시 + 자연어 자동 |
| `/kgulec:setup` | D 셋업 | 색 테마 + 공통 설정(교수명·소속·언어·강의시간) | **명시 전용** |

A/B/C는 "강의자료 만들어줘" 같은 자연어로도 자동 트리거된다. `setup`은 `/kgulec:setup`으로만 실행된다.

## 설치 (마켓플레이스)

```
/plugin marketplace add <github-owner>/<repo>
/plugin install kgulec@kgulec-marketplace
```

이후 처음 한 번 `/kgulec:setup`으로 테마·공통 설정을 잡아두면, 이후 생성물에 자동 적용된다.

## 사용자 설정 (업데이트 안전)

공통 설정은 **플러그인 폴더 바깥** `~/.claude/kgulec/config.md`에 저장된다. 플러그인을 업데이트(`/plugin update`)해도 설정은 유지된다.

- 저장: 교수명·소속·기본 언어·기본 강의시간·색 테마(이름 + 해석된 4색).
- 스키마가 올라가 새 항목이 생기면, 다음 실행 때 **기존 값은 유지한 채 새 항목만** 자동 보강(필요 시 그것만 질문)한다. 스키마 정의는 `shared/config-schema.md`.

## 색 테마

`/kgulec:setup`에서 주색(primary) 프리셋을 고른다. 의미색(example=초록, warning=주황, info=회색)은 고정이고 **주색만** 바뀐다. 기본 6종: Ocean(파랑·기본) / Indigo / Plum / Maroon / Slate / Teal. 카탈로그는 `shared/color-presets.md`.

## 디렉토리 구조

```
kgulec/
  .claude-plugin/
    plugin.json                 매니페스트 (name: kgulec)
    marketplace.json            배포용 마켓플레이스 정의
  skills/
    new/SKILL.md                모드 A: 생성 (config 로드·오버레이 포함)
    edit/SKILL.md               모드 B: 수정·보완
    code/SKILL.md               모드 C: 실습코드
    setup/SKILL.md              모드 D: 셋업 (수동 전용)
  shared/                       ${CLAUDE_PLUGIN_ROOT}/shared 로 참조
    beamer-templates.md         표준 프리앰블·패턴 (PRIMARY THEME 블록)
    slide-authoring-rules.md    디자인·분량·오버플로우·주의 (A·B 공통)
    color-presets.md            색 프리셋 카탈로그
    config-schema.md            사용자 설정 스키마 + 마이그레이션 규칙
    gotcha/                     반복 실수 방지 기록
  assets/
    kyonggi_logo.png            경기대 로고 (상표, 사용자 배치)
    README.md
  README.md  LICENSE
```

## 요구사항 (LaTeX)

컴파일에는 TeX 배포판(권장 **TeX Live full**)과 다음 패키지가 필요하다(`shared/beamer-templates.md` 기준):

- `beamer`(Madrid 테마), `tcolorbox`(`most`), `kotex`(한글), `fontawesome5`, `algorithm2e`
- `amsmath`, `tikz`, `graphicx`, `booktabs`, `array`, `multicol`, `multirow`
- `xcolor`(`svgnames,table`), `csquotes`, `bbm`, `pifont`, `ragged2e`, `fp`, `hyperref`, `verbatim`

도구: 컴파일 `pdflatex`/`latexmk`, 오버플로우 자동 검사 `pdftoppm`(poppler).

## 경기대 로고

타이틀 로고는 `assets/kyonggi_logo.png`(파일명 고정 — 템플릿 `\titlegraphic`이 찾음). 생성 시 출력 디렉토리로 복사된다. 없어도 `\IfFileExists`라 컴파일은 안 깨진다. 자세한 내용은 `assets/README.md`.

## 라이선스

[MIT License](LICENSE) — © 2026 서왕덕.

단, **경기대학교 로고(`assets/kyonggi_logo.png`)는 경기대학교 상표로 MIT 적용 대상이 아니다.** 다른 곳에서 재사용할 때는 로고 파일을 제거하고 각자의 로고로 교체한다.
