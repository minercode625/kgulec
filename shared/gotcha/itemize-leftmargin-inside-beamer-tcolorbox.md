# Beamer + tcolorbox 안 `\begin{itemize}[leftmargin=*]` 가 beamer item 파서를 깨뜨림

## 증상

Beamer 프레임 안의 tcolorbox에서 `\begin{itemize}[leftmargin=*]`을 사용하면 컴파일 시 다음 에러 발생:

```
! Use of \beamer@parseitem doesn't match its definition.
\beamer@defaultospec ->leftmargin=*
l.758 \end{frame}
!  ==> Fatal error occurred, no output PDF file produced!
```

에러 라인이 `\end{frame}`을 가리키지만, 실제 원인은 그 안의 `itemize`의 옵션이다.

## 원인

Beamer는 `itemize`/`enumerate`의 대괄호 옵션을 **item마다의 출력 템플릿(예: `[<+->]`, `[\bullet]`)** 으로 해석한다. 반면 표준 LaTeX에서 `\begin{itemize}[leftmargin=*]`은 `enumitem` 패키지의 기능이다. Beamer 환경에서 이 옵션이 들어오면 beamer 파서가 `leftmargin=*`을 item 템플릿으로 잘못 해석해 충돌한다.

```latex
% 에러 발생!
\begin{tcolorbox}[warningbox={...}]
  \scriptsize
  \begin{itemize}[leftmargin=*]
    \item 첫 번째
    \item 두 번째
  \end{itemize}
\end{tcolorbox}
```

## 해결 방법

1. **`[leftmargin=*]` 옵션 제거** (가장 간단):
   ```latex
   \begin{itemize}
     \item 첫 번째
   \end{itemize}
   ```
2. 들여쓰기를 줄이고 싶다면 **프리앰블에서 `\leftmargini`를 조정**한다 (이미 표준 프리앰블에 `\setlength{\leftmargini}{1.1em}` 포함됨).
3. 꼭 옵션이 필요하면 `\usepackage{enumitem}`을 명시적으로 로드해야 하며, beamer와의 호환성 문제가 있어 권장하지 않는다.

## 일반 규칙

Beamer 프레임 안에서는 `itemize`/`enumerate`의 대괄호 옵션을 사용하지 않는다. 들여쓰기 조정은 프리앰블의 `\leftmargini`로 일괄 처리한다.
