# 색상 프리셋 카탈로그

`/kgulec:setup` 가 사용자에게 제시하는 **주색(primary) 프리셋** 목록이다. 각 프리셋은 주색 한 가지의 4단계 램프(`base`/`deep`/`mid`/`light`)로 구성된다. 의미색(example=초록, warning=주황, info=회색)은 프리셋과 무관하게 항상 고정이다.

## 적용 방법

선택된 프리셋의 4색을 `shared/beamer-templates.md`의 `% === PRIMARY THEME ===` ~ `% === END PRIMARY THEME ===` 블록에 RGB로 치환한다(매핑: base→`mqblue`, deep→`mqdeepblue`, mid→`mqgrayblue`, light→`mqlightblue`). `conceptbg`/`conceptfr`는 `\colorlet`으로 주색을 자동 추종하므로 따로 건드리지 않는다.

> setup은 정본 템플릿을 직접 고치지 않고, 선택값을 **사용자 config**(`~/.claude/kgulec/config.md`)의 `theme`/`theme_colors`에 저장한다. 모드 A(`/kgulec:new`)가 생성 시 그 값으로 PRIMARY THEME 4줄을 치환한다.

## 프리셋 (초록·주황 계열 제외 — 의미색과 충돌 방지)

| 이름 | base | deep | mid | light | 성격 |
|------|------|------|-----|-------|------|
| **Ocean** | `91,155,213` | `47,85,151` | `79,129,189` | `221,235,247` | 파랑(기본·현행) |
| **Indigo** | `92,107,192` | `40,53,147` | `63,81,181` | `225,227,245` | 남보라, 차분 |
| **Plum** | `156,102,188` | `94,53,128` | `130,80,160` | `238,228,245` | 보라, 강조감 |
| **Maroon** | `178,76,84` | `122,32,42` | `158,54,64` | `244,226,228` | 적갈, 클래식 |
| **Slate** | `96,125,154` | `49,69,91` | `79,103,130` | `226,232,238` | 청회색, 무채 톤 |
| **Teal** | `38,166,154` | `17,94,89` | `30,130,122` | `220,240,238` | 청록(example 초록과 구분) |

### 스와치(대략적 hex, 사용자 제시용)

- Ocean — base `#5B9BD5`, deep `#2F5597`
- Indigo — base `#5C6BC0`, deep `#283593`
- Plum — base `#9C66BC`, deep `#5E3580`
- Maroon — base `#B24C54`, deep `#7A202A`
- Slate — base `#607D9A`, deep `#31455B`
- Teal — base `#26A69A`, deep `#115E59`

## 새 프리셋 추가 시

- 이 표에 한 행(이름 + 4색) 추가.
- 주색 hue가 example 초록(`56,118,29`)·warning 주황(`191,100,0`)과 시각적으로 겹치지 않도록 한다.
- `light`는 배경용 옅은 틴트(명도 높고 채도 낮게) — `darktext(45,45,45)` 본문이 잘 보여야 한다.
