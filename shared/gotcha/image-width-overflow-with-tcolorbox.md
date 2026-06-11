# 이미지 + tcolorbox 조합에서 width= 사용 시 세로 오버플로

## 증상

`\includegraphics[width=0.78\textwidth]{...}` 처럼 가로폭 기반으로 이미지를 삽입한 뒤 같은 프레임 안에 tcolorbox를 추가하면 다음과 같은 경고가 발생한다:

```
Overfull \vbox (54.26141pt too high) detected at line 630
Overfull \vbox (77.19633pt too high) detected at line 1514
```

이미지 자체는 잘려 보이지 않을 수 있지만, footline과 겹치거나 슬라이드 영역을 벗어난다.

## 원인

이미지의 원본 비율이 가로:세로 ≈ 4:3 ~ 1:1 정도인 경우, `width=0.78\textwidth` (약 9.5cm)로 설정하면 세로가 자동으로 약 5~7cm가 된다. 거기에 frametitle (~0.8cm) + 본문 텍스트 + tcolorbox (~1.2cm) + vspace 들이 합쳐지면 7cm 콘텐츠 한도를 초과한다.

`width=`로만 지정하면 LaTeX이 세로를 비례적으로 키우고, 자동 축소가 일어나지 않는다.

## 해결 방법

**같은 프레임에 tcolorbox나 다른 텍스트 블록이 함께 있을 때는 이미지를 `height=`로 명시**한다. 4cm 내외가 안전 범위.

```latex
% 위험: 가로폭만 지정 — 이미지 세로가 5~7cm로 커져 오버플로
\begin{frame}{Mini-batch in Practice}
  \textbf{One epoch = passing through all mini-batches once.}
  \begin{center}
    \includegraphics[width=0.78\textwidth]{./figs/batch_alg.png}
  \end{center}
  \begin{tcolorbox}[infobox={\faIcon{redo} Epoch}]
    ...
  \end{tcolorbox}
\end{frame}

% 안전: 세로를 명시 (4.0~4.4cm)
\begin{frame}{Mini-batch in Practice}
  \textbf{One epoch = passing through all mini-batches once.}
  \begin{center}
    \includegraphics[height=4.0cm]{./figs/batch_alg.png}
  \end{center}
  \begin{tcolorbox}[infobox={\faIcon{redo} Epoch}]
    ...
  \end{tcolorbox}
\end{frame}
```

## 일반 규칙

- 이미지만 단독으로 있는 프레임 → `width=0.86\textwidth` 정도까지 OK
- 이미지 + tcolorbox/텍스트 블록이 같이 있는 프레임 → `height=4.0~4.4cm` 권장
- 이미지 + 두 개 이상의 보조 블록 → `height=3.4~3.8cm`로 더 보수적으로
- 가로폭 기반(`width=`)을 쓰고 싶으면 이미지 원본 비율을 확인 (가로형 16:9 정도면 `width=0.86`도 안전)
