# matplotlib mathtext 는 LaTeX 축약을 모른다 (그림 생성 스크립트가 죽는다)

`mkfigs.py` 등에서 라벨·제목에 LaTeX 습관대로 쓰면 그림 생성이 **예외로 중단**된다.
그 시점까지 만든 그림만 남아, 뒤쪽 그림이 없는 채로 컴파일에 들어가면
`\includegraphics` 가 파일을 못 찾는다.

## `\frac12` 같은 인수 축약 → `ParseSyntaxException`
**증상** `ValueError: ... ParseSyntaxException: Expected \frac{num}{den}, found '12' (at char 9)`.
스택 끝이 `_mathtext.py ... parse` 다.
**원인** mathtext 파서는 `\frac12`(중괄호 없는 단일 토큰 인수) 를 지원하지 않는다.
TeX 본문에서는 되지만 mathtext 에서는 안 된다.
**해결** 인수를 항상 중괄호로: `r"$\frac{1}{2}(x^2+20y^2)$"`.

## 그 밖에 mathtext 가 못 받는 것
- `\begin{bmatrix}` 등 환경 → 행렬은 평문으로 `$[[1,1],[0,1]]$`
- `\text{...}` 는 되지만 `\mathbb{1}`·`\bm` 등 패키지 명령은 안 된다 → `$\mathbf{1}$`
- `\big`·`\Big`·`\left\|` 등 **크기 조절 명령** → `ParseFatalException: Unknown symbol: \big`.
  `\|`·`\left(`\ 는 되지만 `\big\|`·`\Big(` 는 안 된다 → 그냥 `\|`·`(` 로 쓴다
- 한글은 폰트가 없어 두부(□)로 나온다 → 그림 라벨은 전부 영어

**일반 규칙** 그림 스크립트를 처음 돌릴 때 `wrote ...` 줄 개수가 함수 개수와 같은지 확인한다.
중간에 죽으면 개수가 모자란다.
