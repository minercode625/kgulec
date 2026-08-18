# 선택인수 대괄호가 바깥 명령의 대괄호와 충돌한다

## `\item[\faIcon[regular]{square}]` → `Argument of \faIcon has an extra }`
**증상** `! Argument of \faIcon has an extra }.` 와
`! Paragraph ended before \faIcon was complete.` 가 번갈아 쏟아지고 에러가 `\end{frame}`을 가리킨다.
`\faIcon{square}`(선택인수 없음)는 멀쩡한데 `[regular]`를 붙이는 순간 깨진다.
**원인** `\item[...]`은 첫 `]`까지를 라벨로 삼는다. 안쪽 `\faIcon[regular]`의 `]`가 먼저 걸려
라벨이 `\faIcon[regular`에서 끊기고, 남은 `{square}]`가 본문으로 흘러간다.
**해결** 선택인수를 가진 명령을 대괄호 인수 안에 넣을 때는 **중괄호로 한 번 더 감싼다.**
```latex
% 위험
\item[\faIcon[regular]{square}] 항목
% 안전
\item[{\faIcon[regular]{square}}] 항목
```
**일반 규칙** `\item[...]`·`\caption[...]`·`\\[...]` 등 대괄호 인수 안에 `[`가 나오면
전체를 `{}`로 감싼다. 번들 `short-arg-linebreak.md`의 「`\\` 다음이 `[`」와 같은 부류다.
빈 체크박스가 필요하면 `\faIcon[regular]{square}`가 맞다 --- 기본(solid)은 **채워진** 사각형이라
체크리스트로 쓰면 이미 체크된 것처럼 보인다.
