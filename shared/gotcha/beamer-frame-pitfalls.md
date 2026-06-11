# Beamer 프레임 내부 함정 (verbatim · itemize 옵션)

## verbatim / `#` 포함 프레임 → `[fragile]` 필수
**증상** `Illegal parameter number in definition of \iterate`, `Paragraph ended before \verbatim@ was complete`, 심하면 `\begin{frame} ended by \end{document}`(PDF 생성 실패).
**원인** beamer는 프레임 내용을 매크로 인수로 처리 → verbatim의 catcode 변경, `#`과 충돌.
**해결** verbatim(또는 `#`) 든 프레임엔 `\begin{frame}[fragile]`. tcolorbox 안 verbatim도 동일.

## `\begin{itemize}[leftmargin=*]` (beamer tcolorbox 안) → 파서 깨짐
**증상** `Use of \beamer@parseitem doesn't match its definition` → Fatal, PDF 없음. 에러는 `\end{frame}` 가리킴.
**원인** beamer는 itemize/enumerate 대괄호 옵션을 item 템플릿(`[<+->]` 등)으로 해석. `leftmargin=*`(enumitem 기능)과 충돌.
**해결** 옵션 제거. 들여쓰기는 프리앰블 `\leftmargini`로(표준 프리앰블에 `\setlength{\leftmargini}{1.1em}` 있음). beamer 프레임 안 itemize/enumerate엔 대괄호 옵션 쓰지 말 것.
