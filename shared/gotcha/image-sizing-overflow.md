# 이미지 크기 지정 실수로 인한 오버플로 (width/height 선택)

`\includegraphics`에서 한 축만 지정하면 다른 축이 원본 비율로 자동 확대돼 슬라이드를 넘는다. 컴파일 에러 없이(Overfull 경고만, 때론 그것도 없이) 이미지가 footline을 가리거나 잘린다. **종횡비 + 같은 프레임의 다른 콘텐츠**로 축을 고른다.

## 가로형(비율 > 2:1)에 `height=` → 가로 초과
**증상** `Overfull \hbox (...too wide)`. 예: 950x177(≈5.4:1)에 `height=4cm` → 가로 약 21cm로 슬라이드(≈12.8cm) 우측 잘림.
**해결** 가로≫세로(>2:1)는 `width=`. `width=0.92\textwidth` → 가로 ≈11.8cm.

## 세로형(portrait)에 `width=\linewidth` (columns) → 세로 초과
**증상** 좁은 column에 인물 사진을 `width=\linewidth` → 세로가 비례 확대돼 캡션이 footline과 겹침.
**해결** 세로≫가로는 `height=`. columns 안 인물 사진은 `height=2.4~2.8cm`(보조 블록 더 있으면 2.2cm). 나란히 둘 땐 같은 height로 통일.

## 정사각~4:3 + tcolorbox/텍스트 → 세로 초과
**증상** `width=0.78\textwidth`로 넣고 같은 프레임에 박스 추가 → `Overfull \vbox (...too high)`, footline 침범.
**해결** 같은 프레임에 박스/텍스트가 있으면 `height=` 명시: 박스 1개 `height=4.0~4.4cm`, 2개 이상 `3.4~3.8cm`. 이미지 단독이면 `width=0.86\textwidth`까지 OK.

## 일반 규칙
- 작업 전 `identify` / `sips -g pixelWidth -g pixelHeight`로 원본 비율 확인.
- 가로≫세로(>2:1) → `width=`. 세로≫가로 → `height=`. 정사각~4:3 → 단독이면 무엇이든, 박스와 함께면 `height=`.
- 콘텐츠 세로 한도 ≈7cm. height 지정 시 가로폭이, width 지정 시 세로높이가 한도를 넘는지 환산 확인.
