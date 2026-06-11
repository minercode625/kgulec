# 프레임 제목(또는 movable arg) 안의 `&`가 "Misplaced alignment tab" 에러

## 증상

컴파일이 다음 에러로 실패한다. 에러 라인은 엉뚱하게 `\end{frame}`을 가리킨다:

```
! Misplaced alignment tab character &.
<recently read> &
l.840 \end{frame}
```

## 원인

`\begin{frame}{... A & B ...}` 처럼 **프레임 제목(frametitle)** 안에 이스케이프하지 않은 `&`를 쓰면, LaTeX이 이를 표(`tabular`)의 열 구분자(alignment tab)로 해석한다. 프레임 제목은 movable argument라 더 쉽게 깨진다.

```latex
% 에러 발생!
\begin{frame}[fragile]{터미널에서 실행 & 알아두면 좋은 것}
```

에러 메시지는 그 프레임의 `\end{frame}` 줄을 가리키지만, 진짜 원인은 제목의 `&`다.

## 해결 방법

제목·본문 텍스트의 앰퍼샌드는 항상 `\&`로 이스케이프한다.

```latex
% 안전
\begin{frame}[fragile]{터미널에서 실행 \& 알아두면 좋은 것}
```

## 일반 규칙

- `tabular`/`matrix`의 열 구분용 `&`만 raw로 두고, 그 외 모든 텍스트(제목, 박스 타이틀, 본문, `\section{...}`)의 `&`는 `\&`로 쓴다.
- `\section{실전 팁 \& 막혔을 때}`, `\sectiondividerframe{... \& ...}` 도 동일.
