# Beamer frame에서 verbatim 사용 시 [fragile] 필수

## 증상

`verbatim` 환경을 포함한 프레임에서 다음과 같은 에러 발생:
- `! Illegal parameter number in definition of \iterate.`
- `! Paragraph ended before \verbatim@ was complete.`
- `! You can't use 'macro parameter character #' in internal vertical mode.`
- 심한 경우 `\begin{frame}` ended by `\end{document}` 에러로 PDF 생성 실패

## 원인

Beamer는 프레임 내용을 내부적으로 매크로 인수로 처리하는데, `verbatim` 환경은 catcode를 변경하므로 충돌이 발생한다. `#` 문자도 동일한 이유로 문제를 일으킨다.

## 해결

`verbatim` 환경이 포함된 모든 프레임에 `[fragile]` 옵션을 추가한다:

```latex
% Bad
\begin{frame}{제목}
  \begin{verbatim}
  코드 내용
  \end{verbatim}
\end{frame}

% Good
\begin{frame}[fragile]{제목}
  \begin{verbatim}
  코드 내용
  \end{verbatim}
\end{frame}
```

**주의**: `tcolorbox` 안에 `verbatim`을 넣는 경우에도 반드시 `[fragile]` 필요.
