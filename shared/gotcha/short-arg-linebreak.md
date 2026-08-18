# short 명령 인수 · `\\` 줄바꿈 파싱 함정

둘 다 에러가 `\end{frame}`을 가리키지만 진짜 원인은 프레임 내부다.

## `\par`/`\\`를 `\textbf{...}` 등 short 명령 인수에 넣음
**증상** `Paragraph ended before \text@command was complete.` → `Missing \endgroup/} inserted`, `Misplaced \cr` 연쇄. (tabular 셀 `\textbf{Supervised\par(지도)}` 식)
**원인** `\textbf`(\textit/\emph/\textsf 등 short 명령)은 인수에 단락 끝(`\par`/빈 줄)·`\\`를 못 받음.
**해결** 줄바꿈을 명령 밖으로: `\textbf{Supervised}\\(지도)`, 또는 `\makecell{\textbf{A}\\B}`(makecell 패키지).

## `\\` 다음이 `[`로 시작
**증상** `! Illegal unit of measure (pt inserted).` `\\` 뒤 `[역할]` 같은 대괄호 라벨이 `\\[길이]` 선택인수로 파싱됨.
**해결** `\\` 뒤에 빈 그룹: `[역할] ...\\{}`. 의도적 간격일 때만 `\\[0.3cm]`. `\item` 등 `\\` 받는 환경 + 대괄호 라벨에서도 동일.

## `m{}`/`p{}` 셀 안의 `\\` 는 줄바꿈이 아니라 **행 끝**이다
**증상** 에러·경고 없음. 표의 한 칸이 두 행으로 쪼개져 뒤 조각이 다음 행 **1열**에 나타난다
(`FID & 분포 거리\\(낮을수록 좋음) & ...` -> 다음 줄 1열에 `(낮을수록 좋음)`). 시각 검사로만 발견.
**원인** `tabular` 안의 `\\` 는 언제나 행 종료다. `m{}`/`p{}` 가 자동 줄바꿈을 해 주므로 헷갈리기 쉽다.
**해결** 한 줄로 두고 자동 줄바꿈에 맡기거나, 꼭 끊어야 하면 `\newline`(`m{}`/`p{}` 에서만 동작)을 쓴다.
**점검** 표를 넣은 프레임은 렌더해서 행 수가 의도대로인지 센다.
