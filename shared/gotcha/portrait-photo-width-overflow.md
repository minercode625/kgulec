# 좁은 column에 세로형 사진을 width=\linewidth로 넣으면 footline 가림

## 증상

`columns` 환경 안에서 세로 비율이 큰 인물 사진(440x587 등 portrait)을 `\includegraphics[width=\linewidth]`로 넣으면, column 폭에 맞춰 가로가 커지면서 **세로가 비례해 더 커진다**. 결과적으로 사진 + 캡션이 슬라이드 하단의 footline과 겹친다.

검사에서 "사진 캡션이 footline의 author/title 영역과 정확히 겹친다"로 보고된다.

## 원인

`columns`의 column 폭이 좁아도(예: `0.36\textwidth`), `width=\linewidth`는 그 폭을 채우라고 지시한다. 가로:세로 비율이 1:1.3 이상인 인물 사진은 가로 4cm가 되면 세로가 5.2cm 이상이 되고, 그 아래 캡션과 합치면 7cm 콘텐츠 한도를 넘어버린다.

## 해결 방법

**세로 비율이 큰 사진은 `width=` 대신 `height=`로 크기를 지정한다.**

```latex
% 위험: 좁은 column에 portrait 사진을 width로 넣음
\column{0.36\textwidth}
  \includegraphics[width=\linewidth]{figs/dijkstra.jpg}  % 세로가 컬럼 폭에 비례해 커짐
  \begin{center}\scriptsize 캡션\end{center}

% 안전: height로 명시 (보통 2.4~2.8cm)
\column{0.30\textwidth}
  \begin{center}
    \includegraphics[height=2.6cm]{figs/dijkstra.jpg}\\
    \scriptsize 캡션
  \end{center}
```

## 일반 규칙

- 세로형 인물 사진(portrait)을 columns 안에 배치할 때는 **`height=2.4cm ~ 2.8cm`** 가 안전 범위
- 사진 + 캡션 + 다른 박스가 한 슬라이드에 있을 땐 더 보수적으로(`height=2.2cm`)
- 가로형 사진이나 도표는 `width=\linewidth` OK
- 같은 슬라이드에 여러 인물 사진을 나란히 둘 때 모두 같은 `height=`로 통일하면 정렬도 깔끔
