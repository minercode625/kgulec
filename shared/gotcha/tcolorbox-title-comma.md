# tcolorbox 타이틀 내 쉼표가 tcb 키로 파싱되는 문제

## 증상

컴파일 시 `Package pgfkeys Error: I do not know the key '/tcb/...'` 에러 발생.
예: `I do not know the key '/tcb/제4국'`, `I do not know the key '/tcb/$n_C=2$'`

## 원인

`\begin{tcolorbox}[examplebox={타이틀 내용}]`에서 타이틀 안에 **쉼표(`,`)**가 있으면, LaTeX이 이를 tcolorbox 옵션의 키-값 구분자로 해석한다.

문제 예시:
```latex
% 에러 발생!
\begin{tcolorbox}[conceptbox={\faIcon{star} 2016년 3월 13일, 제4국}]
\begin{tcolorbox}[examplebox={\faIcon{search} 라운드 6: C에서 보상 0. $N=5$, $n_C=2$}]
\begin{tcolorbox}[conceptbox={\faIcon{globe} 하나의 알고리즘, 세 게임}]
```

## 해결 방법

1. **쉼표 제거**: 문장을 쉼표 없이 재구성 (가장 간단)
2. **`{,}` 사용**: 쉼표를 중괄호로 감싸기 — `하나의 알고리즘{,} 세 게임`
3. **수식 내 쉼표**: `$N=5$\,/\,$n_C=2$` 처럼 쉼표 대신 슬래시 등 다른 구분자 사용

```latex
% 수정 예시
\begin{tcolorbox}[conceptbox={\faIcon{star} 2016년 3월 13일 제4국}]
\begin{tcolorbox}[examplebox={\faIcon{search} 라운드 6: C에서 보상 0 ($N{=}5$\,/\,$n_C{=}2$)}]
\begin{tcolorbox}[conceptbox={\faIcon{globe} 하나의 알고리즘{,} 세 게임}]
```
