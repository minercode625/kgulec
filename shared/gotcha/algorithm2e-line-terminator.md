# algorithm2e 안에서 `\;` 는 공백이 아니라 줄 끝이다

**증상** 컴파일 에러는 없다. 그런데 의사코드 한 줄이 여러 줄로 쪼개지고
`m ← 0, ;` 처럼 **쉼표 뒤에 세미콜론만 남은 줄**이 생긴다. PDF 시각 검사로만 발견된다.

**원인** `algorithm2e` 는 `\;` 를 **문장 종료**로 재정의한다(끝에 `;` 를 찍고 줄을 넘긴다).
수식 사이 간격을 주려고 `\;` 를 쓰면 그 자리에서 줄이 끊긴다.

```latex
% 위험 -- 세 개의 반쪽 줄이 된다
$\mathbf{m} \leftarrow \mathbf{0}$, \; $\mathbf{v} \leftarrow \mathbf{0}$, \; $t \leftarrow 0$\;

% 안전 -- 간격은 \quad / \, 로, 줄 끝에만 \;
$\mathbf{m} \leftarrow \mathbf{0}$, \quad $\mathbf{v} \leftarrow \mathbf{0}$, \quad $t \leftarrow 0$\;
```

**해결** algorithm 환경 안에서 간격은 `\quad`·`\qquad`·`\,` 로 주고, `\;` 는 **줄 맨 끝에만** 쓴다.
`\tcp{주석}` 은 줄 끝 주석이므로 그 뒤에는 `\;` 를 붙이지 않는다.
