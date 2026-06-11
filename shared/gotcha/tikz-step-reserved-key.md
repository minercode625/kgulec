# TikZ 스타일명 `step`이 예약어와 충돌

## 증상

컴파일 시 `Package pgfkeys Error: The key '/tikz/step' requires a value.` 에러 발생.

## 원인

TikZ에서 `step`은 이미 정의된 키(예: `xstep`, `ystep` 등의 축약)이다. 따라서 사용자 정의 노드 스타일 이름으로 `step`을 사용하면 충돌이 발생한다.

```latex
% 에러 발생!
\begin{tikzpicture}[
    step/.style={draw, rounded corners=5pt, ...}]
    \node[step, draw=mqblue] (sel) {...};
```

## 해결 방법

`step` 대신 다른 이름을 사용한다. 예: `sbox`, `stepbox`, `phasebox` 등.

```latex
% 수정
\begin{tikzpicture}[
    sbox/.style={draw, rounded corners=5pt, ...}]
    \node[sbox, draw=mqblue] (sel) {...};
```

## 일반 규칙

TikZ 스타일 이름으로 다음 예약어를 피한다:
- `step`, `shift`, `scale`, `rotate`, `color`, `fill`, `draw`, `node`, `path`, `edge`, `out`, `in`
- (`in`, `out`은 `to[in=...,out=...]` 같은 베지어 곡선 옵션 예약어다.)
- 안전한 패턴: 접두사를 붙이기 (예: `mybox`, `sbox`, `phasebox`, `inp`, `outp`)
