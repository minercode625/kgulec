# 사용자 설정 스키마 (config schema)

`/kgulec:setup` 가 쓰고 `/kgulec:new`(모드 A)가 읽는 **사용자 상수 설정**의 단일 정의다.

- 설정 파일 경로: **사용자 홈의 `.claude/kgulec/config.md`** (플러그인 폴더 바깥 → 플러그인 업데이트가 덮어쓰지 않음). macOS/Linux `~/.claude/kgulec/config.md`, Windows `%USERPROFILE%\.claude\kgulec\config.md`.
- 이 스키마 파일은 버전관리 대상(읽기전용). config 파일은 비버전(사용자 소유).

## 현재 스키마 버전

```
schema_version: 1
```

## 필드 정의

| 필드 | 타입 | 기본값 | 설명 |
|------|------|--------|------|
| `config_version` | int | (저장 시 `schema_version` 값) | 이 config가 따르는 스키마 버전 |
| `professor` | string | `"교수명"` | 강의자 이름 → 템플릿 `\professor` |
| `department` | string | `"소속"` | 학과/학부/학교 → 템플릿 `\department` (타이틀 institute 영역) |
| `language` | enum `ko`\|`en` | `ko` | 기본 슬라이드 언어 (사용자가 명시 지정 시 덮어씀) |
| `lecture_minutes` | int | `180` | 기본 강의 길이(분, 3시간) → 분량 가이드의 슬라이드 수 기준. **모드 A는 생성 시 이번 강의 시간을 확인하고 특강 등은 override한다**(이번 강의 한정, config는 덮어쓰지 않음) |
| `theme` | string | `"Ocean"` | 선택한 색 프리셋 이름 (`color-presets.md` 참조) |
| `theme_colors` | object | Ocean 값 | 프리셋의 해석된 4색. **이름이 아니라 이 값이 권위 있음**(카탈로그가 바뀌어도 사용자 슬라이드 안 흔들림) |

`theme_colors` 형식:
```yaml
theme_colors:
  base:  "91,155,213"
  deep:  "47,85,151"
  mid:   "79,129,189"
  light: "221,235,247"
```

## config.md 예시

```yaml
config_version: 1
professor: 홍길동
department: 경기대학교 컴퓨터공학부
language: ko
lecture_minutes: 180
theme: Ocean
theme_colors:
  base:  "91,155,213"
  deep:  "47,85,151"
  mid:   "79,129,189"
  light: "221,235,247"
```

## 읽기 규칙 (모드 A·D 공통)

1. config가 없으면 → setup을 먼저 안내(또는 모든 값을 기본값으로 가정하고 진행, 강의별 항목만 질문).
2. 있으면 **defaults ⊕ config 병합**: 누락 키는 위 표의 기본값으로 채운다(절대 크래시 금지).
3. config에 있는 값은 모드 A에서 재질문하지 않는다(강의별 항목만 질문: 과목명·주제·주차·날짜).

## 마이그레이션 규칙 (스키마 드리프트 처리)

config는 git 비추적이므로 머지 충돌은 없다. 코드 업데이트로 `schema_version`이 올라가면 다음과 같이 **탑업 마이그레이션**한다(모드 A·D 진입 시 자동):

1. `config.config_version < schema_version` 감지.
2. 저장 전 백업: `config.md` → `config.md.bak`.
3. 새 필드 채우기:
   - 안전한 기본값이 있으면 → 조용히 채우고, 사용자에게 한 줄 통지("새 설정 X 추가됨, 기본값 Y. 바꾸려면 /kgulec:setup").
   - 기본값이 부적절한 정체성 값(예: 새 식별 정보)이면 → **그 필드만** 사용자에게 질문(기존 값은 묻지 않음).
4. 사용자가 임의로 넣은 미지의 키는 보존한다(비파괴).
5. `config_version`을 `schema_version`으로 올려 다시 저장.

## 스키마 변경 규칙 (가산적)

- 새 필드는 항상 optional + 기본값 보유.
- 필드 이름변경/삭제 시 마이그레이션 매핑을 이 문서에 함께 기록.
- `schema_version`을 1 올리고, 위 표·예시·마이그레이션 절을 갱신한다.
