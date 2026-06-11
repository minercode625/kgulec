# TikZ 함정 (예약 스타일명 · overlay 배경)

## 스타일 이름이 예약 키와 충돌
**증상** `Package pgfkeys Error: The key '/tikz/step' requires a value.`
**원인** `step`(=xstep/ystep 축약) 같은 예약어를 사용자 `.style` 이름으로 씀.
**해결** 접두사 붙은 다른 이름: `sbox`/`phasebox`/`mybox`. 피할 예약어: `step, shift, scale, rotate, color, fill, draw, node, path, edge, in, out`(`in/out`은 베지어 `to[in=,out=]`).

## `remember picture,overlay` 배경이 일부 페이지서 깨짐
**증상** `sectiondividerframe` 등 전체배경 fill이 뒤쪽 표지에서 위로 밀려 하단 흰색.
**원인** overlay를 `frame` 본문에 두면 `current page` 앵커가 내용 높이에 따라 흔들림(다회 컴파일해도 미수렴).
**해결** overlay를 `\setbeamertemplate{background canvas}` 안으로 이동(배경을 내용과 독립 렌더).
```latex
{%
\setbeamertemplate{background canvas}{%
  \begin{tikzpicture}[remember picture,overlay]
    \fill[mqdeepblue] (current page.north west) rectangle (current page.south east);
  \end{tikzpicture}%
}%
\begin{frame}[plain] ... \end{frame}
}%
```
