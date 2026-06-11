# 가로로 매우 긴 이미지를 height=로 지정하면 가로폭이 슬라이드를 초과

## 증상

`Overfull \hbox (272.27188pt too wide) in paragraph` — 무려 9.6cm 이상 슬라이드 가로폭을 넘어버린다. 경고만 보고 무시하면 PDF가 컴파일은 되지만, 이미지가 슬라이드 우측 영역을 뚫고 잘려나간다.

## 원인

`\includegraphics[height=4cm]`처럼 **세로만** 지정하면, 가로는 이미지 원본 비율에 비례해 자동 계산된다. 가로:세로 비율이 5:1, 6:1 등으로 극단적으로 큰 이미지(파이프라인 다이어그램, LSTM 셀 횡단면, 시퀀스 모델 unrolling 도식 등에서 흔함)는 `height=4cm`에 가로 20cm 이상이 되어 슬라이드(약 12.8cm)를 크게 초과한다.

```latex
% 위험: lstm2.png는 950x177 (가로:세로 ≈ 5.37:1)
\includegraphics[height=4.0cm]{./figs/lstm2.png}
% → 가로 21.5cm로 렌더링되어 슬라이드 오른쪽이 잘림
```

## 해결 방법

**가로:세로 비율이 2:1보다 큰 이미지는 `width=`로 지정**하고, 같은 프레임에 다른 콘텐츠가 있다면 비율로 환산한 세로 높이를 미리 계산해서 7cm 한도를 넘지 않는지 확인한다.

```latex
% 안전
\includegraphics[width=0.92\textwidth]{./figs/lstm2.png}
% → 가로 ≈ 11.8cm, 세로 ≈ 2.2cm
```

## 일반 규칙

작업 전 `identify` 또는 `sips -g pixelWidth -g pixelHeight`로 이미지 크기를 확인하고:
- **가로 ≫ 세로** (비율 > 2:1) → `width=` 사용
- **세로 ≫ 가로** (portrait) → `height=` 사용 (별도 gotcha: `portrait-photo-width-overflow.md`)
- **거의 정사각형** (1:1 ~ 4:3) → 어느 쪽이든 가능하나, tcolorbox와 함께면 `height=` 권장 (`image-width-overflow-with-tcolorbox.md`)
