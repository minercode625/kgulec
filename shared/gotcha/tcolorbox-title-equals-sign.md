# tcolorbox 타이틀 내 수식의 등호(=)가 key-value 파서와 충돌

## 증상

컴파일 시 `! Missing $ inserted.` 에러 발생. 해당 프레임의 `\end{frame}` 라인에서 보고됨.

## 원인

`\begin{tcolorbox}[examplebox={\faIcon{...} $k = 1$}, ...]`에서 수식 안의 `=`가 tcolorbox 옵션의 key=value 구분자로 해석되어 수식이 깨진다.

```latex
% 에러 발생!
\begin{tcolorbox}[examplebox={\faIcon{search} When $k = 1$}, top=2pt, bottom=2pt]
```

## 해결 방법

수식 내 `=` 를 `{=}`로 감싸서 파서가 키-값 구분자로 인식하지 못하게 한다.

```latex
% 수정
\begin{tcolorbox}[examplebox={\faIcon{search} When $k{=}1$}, top=2pt, bottom=2pt]
```

## 일반 규칙

tcolorbox 타이틀(conceptbox, examplebox, warningbox 등)의 수식 내에서 `=` 사용 시 반드시 `{=}`로 감싼다. 쉼표(`,`)도 마찬가지로 `{,}`로 감싸야 한다 (기존 gotcha 참조).
