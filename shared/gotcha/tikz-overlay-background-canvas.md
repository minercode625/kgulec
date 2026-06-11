# TikZ remember picture,overlay가 일부 섹션 표지에서 배경을 못 채우는 문제

## 증상

`sectiondividerframe`에서 `\begin{tikzpicture}[remember picture,overlay]`로 전체 페이지 배경을 그릴 때, 앞쪽 섹션 표지(예: 1~2번째)는 정상이지만 뒤쪽 섹션 표지에서 파란 배경이 위로 올라가며 하단이 흰색으로 잘린다.

## 원인

`remember picture,overlay` TikZ를 `\begin{frame}[plain]` 본문 안에 넣으면, 프레임 내용의 높이에 따라 `current page` 앵커 위치가 달라질 수 있다. 특히 여러 번 컴파일해도 일부 페이지에서 위치가 수렴하지 않는 경우가 있다.

## 해결 방법

TikZ overlay를 프레임 본문이 아닌 `\setbeamertemplate{background canvas}` 안으로 옮긴다. Beamer가 배경을 프레임 내용과 독립적으로 먼저 렌더링하므로 안정적이다.

```latex
% 잘못된 방식 (일부 페이지에서 배경 깨짐)
\begin{frame}[plain]
\begin{tikzpicture}[remember picture,overlay]
  \fill[mqdeepblue] (current page.north west) rectangle (current page.south east);
  ...
\end{tikzpicture}
\end{frame}

% 올바른 방식
{%
\setbeamertemplate{background canvas}{%
  \begin{tikzpicture}[remember picture,overlay]
    \fill[mqdeepblue] (current page.north west) rectangle (current page.south east);
    ...
  \end{tikzpicture}%
}%
\begin{frame}[plain]
  ... 내용 ...
\end{frame}
}%
```
