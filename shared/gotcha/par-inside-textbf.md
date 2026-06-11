# `\par` 또는 줄바꿈을 `\textbf{...}` 인수에 넣으면 컴파일 실패

## 증상

`tabular` 셀에서 셀 내용을 두 줄로 표시하려고 `\textbf{Supervised\par(지도)}` 처럼 작성하면 다음과 같은 에러가 연쇄 발생:
- `! Paragraph ended before \text@command was complete.`
- `! Missing \endgroup inserted.`
- `! Missing } inserted.`
- `! Misplaced \cr.` (수십~수백 번 반복)
- `! Missing \cr inserted.`

에러 메시지가 `\end{frame}` 라인을 가리키지만, 실제 원인은 그 위 tabular 안의 한 셀이다.

## 원인

`\textbf`는 LaTeX의 `short` 명령으로, 인수에 단락 끝(`\par` 또는 빈 줄)이 들어가면 즉시 실패한다. tabular 셀에서 강제 줄바꿈을 하려고 `\par`를 쓰면 이 제약에 걸린다.

```latex
% 에러 발생!
\begin{tabular}{>{\centering\arraybackslash}m{2.0cm}|...}
  \textbf{Supervised\par(지도)} & ... \\
  \textbf{Unsupervised\par(비지도)} & ...
\end{tabular}
```

## 해결 방법

1. **줄바꿈을 포기하고 같은 줄에 표기** (가장 간단):
   ```latex
   \textbf{Supervised} (지도)
   ```

2. **줄바꿈이 꼭 필요하면 `\textbf` 바깥에 두기** + `\\`:
   ```latex
   \textbf{Supervised}\\(지도)
   ```

3. **`\makecell`/`\shortstack` 사용** (`makecell` 패키지 필요):
   ```latex
   \makecell{\textbf{Supervised}\\(지도)}
   ```

## 일반 규칙

- `tabular` 셀 안에서 `\par`는 사용하지 말고, 줄바꿈은 `\\`로 한다.
- `\\`는 `\textbf{...}` 등 short 명령의 인수 안에서도 쓸 수 없으므로, 줄바꿈이 필요하면 명령 \textbf{바깥}으로 빼거나 `\makecell`을 사용한다.
- 같은 함정이 `\textit`, `\emph`, `\textsf` 등 모든 short 명령에 적용된다.
