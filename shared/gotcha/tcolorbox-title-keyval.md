# tcolorbox 타이틀 안 `,` · `=`가 pgfkeys 키로 파싱됨

`\begin{tcolorbox}[examplebox={타이틀}]`의 타이틀 텍스트에 쉼표·등호가 있으면 tcolorbox 옵션의 키-값 구분자로 해석돼 깨진다.

## 쉼표 `,`
**증상** `Package pgfkeys Error: I do not know the key '/tcb/제4국'`(쉼표 뒤 토막이 키로 해석). 쉼표 뒤 토막이 한글로 시작하면 대신 `! Missing \endcsname inserted.` + `\unih@ngulpoint`로 보고되기도 함(kotex 한글이 `\csname` 키 조회에 들어감).
**해결** `{,}`로 감싸기(`알고리즘{,} 세 게임`), 쉼표 없이 재구성, 또는 수식 구분은 슬래시: `$N{=}5$\,/\,$n_C{=}2$`.
**주의** 이중 중괄호 `examplebox={{...,...}}`도 쉼표를 보호하지 못한다(스타일 `title=#1` 재파싱 시 분리). 반드시 `{,}` 사용.

## 등호 `=`
**증상** `! Missing $ inserted.`(해당 프레임 `\end{frame}`에서 보고). 수식 `$k = 1$`의 `=`가 key=value로 해석.
**해결** `{=}`로 감싸기: `When $k{=}1$`.

## 일반 규칙
conceptbox/examplebox/warningbox 등 타이틀의 수식 안 `=`·`,`는 항상 `{=}`·`{,}`로 감싼다. 타이틀 밖 본문에서는 불필요.
