# short 명령 인수 · `\\` 줄바꿈 파싱 함정

둘 다 에러가 `\end{frame}`을 가리키지만 진짜 원인은 프레임 내부다.

## `\par`/`\\`를 `\textbf{...}` 등 short 명령 인수에 넣음
**증상** `Paragraph ended before \text@command was complete.` → `Missing \endgroup/} inserted`, `Misplaced \cr` 연쇄. (tabular 셀 `\textbf{Supervised\par(지도)}` 식)
**원인** `\textbf`(\textit/\emph/\textsf 등 short 명령)은 인수에 단락 끝(`\par`/빈 줄)·`\\`를 못 받음.
**해결** 줄바꿈을 명령 밖으로: `\textbf{Supervised}\\(지도)`, 또는 `\makecell{\textbf{A}\\B}`(makecell 패키지). 셀 줄바꿈은 `\par` 말고 `\\`.

## `\\` 다음이 `[`로 시작
**증상** `! Illegal unit of measure (pt inserted).` `\\` 뒤 `[역할]` 같은 대괄호 라벨이 `\\[길이]` 선택인수로 파싱됨.
**해결** `\\` 뒤에 빈 그룹: `[역할] ...\\{}`. 의도적 간격일 때만 `\\[0.3cm]`. `\item` 등 `\\` 받는 환경 + 대괄호 라벨에서도 동일.
