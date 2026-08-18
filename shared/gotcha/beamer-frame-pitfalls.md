# Beamer 프레임 내부 함정 (verbatim · itemize 옵션)

## verbatim / `#` 포함 프레임 → `[fragile]` 필수
**증상** `Illegal parameter number in definition of \iterate`, `Paragraph ended before \verbatim@ was complete`, 심하면 `\begin{frame} ended by \end{document}`(PDF 생성 실패).
**원인** beamer는 프레임 내용을 매크로 인수로 처리 → verbatim의 catcode 변경, `#`과 충돌.
**해결** verbatim(또는 `#`) 든 프레임엔 `\begin{frame}[fragile]`. tcolorbox 안 verbatim도 동일.

**verbatim 없이도 걸림 — TikZ 스타일의 `#1`**: 프레임 안에서 인자 받는 스타일을 정의하면 `#`이 들어간다. verbatim이 없어 `[fragile]`을 안 붙이고, 에러는 `\iterate`를 가리켜 TikZ와 무관해 보인다.
```latex
% 위험: \begin{frame}{흐름도} + \begin{tikzpicture}[gbox/.style={draw=#1, fill=#1!8}]
% 안전: \begin{frame}[fragile]{흐름도}   (또는 스타일 인자를 없애고 노드마다 draw= 지정)
```

## `\begin{itemize}[leftmargin=*]` (beamer tcolorbox 안) → 파서 깨짐
**증상** `Use of \beamer@parseitem doesn't match its definition` → Fatal, PDF 없음. 에러는 `\end{frame}` 가리킴.
**원인** beamer는 itemize/enumerate 대괄호 옵션을 item 템플릿(`[<+->]` 등)으로 해석. `leftmargin=*`(enumitem 기능)과 충돌.
**해결** 옵션 제거. 들여쓰기는 프리앰블 `\leftmargini`로(표준 프리앰블에 `\setlength{\leftmargini}{1.1em}` 있음). beamer 프레임 안 itemize/enumerate엔 대괄호 옵션 쓰지 말 것.

## 프레임 제목 바로 뒤의 `{...}` 그룹이 부제로 먹힘 (`[fragile]` 아니어도 발생)

**증상** `! Paragraph ended before \beamer@verbatim@framesubtitle was complete.` 이어서
`You can't use 'macro parameter character #' in horizontal mode`, `Missing $ inserted` 가 verbatim 본문 줄마다 연쇄. verbatim 내용이 본문으로 조판된다.
**원인** `[fragile]` 프레임은 `{제목}{부제}` 를 탐욕적으로 스캔한다. 제목 다음에 오는 첫 토큰이 `{` 면 그 그룹을 **부제로 삼는다** — `{\scriptsize ... \begin{verbatim}` 같은 크기 지정 그룹이 여기 걸린다.
**해결** 제목 뒤 첫 토큰을 `{` 가 아니게 한다. 크기 지정은 그룹 대신 스위치로.
```latex
% 위험
\begin{frame}[fragile]{코드}
  {\scriptsize
\begin{verbatim} ... \end{verbatim}
  }
% 안전
\begin{frame}[fragile]{코드}
  \scriptsize
\begin{verbatim} ... \end{verbatim}
```
(`\vspace{...}` 등 `\` 로 시작하는 토큰을 먼저 두면 막힌다.)

**`[fragile]` 아닌 일반 프레임도 같다.** 이쪽은 에러가 없어 더 위험하다 --- 본문 첫 줄 `{\scriptsize $z^{(l+1)}=\dots$ 이므로 \dots\par}` 가 통째로 **제목 띠 안**에 작게 조판된다. Overfull 경고도 없어 **PDF 렌더 검사로만** 발견된다. 본문을 `{` 로 시작해야 하면 앞에 `\vspace{-0.05cm}` 같은 토큰을 하나 둔다.

## 크기 명령이 `verbatim` **안**에 들어감 → 글자로 찍히고 넘침

**증상** 슬라이드 코드 블록 첫 줄에 `\scriptsize` 가 **코드처럼 그대로 출력**된다.
동시에 블록이 줄어들지 않아 `Overfull \vbox (70pt too high)` 로 footline을 침범한다.
컴파일은 성공하므로 로그만 봐서는 원인이 안 보인다.
**원인** `verbatim` 은 내용을 문자 그대로 출력한다. 크기 지정을 `\begin{verbatim}` **다음 줄**에 두면
명령이 아니라 텍스트가 된다.
```latex
% 잘못 — \scriptsize 가 코드 한 줄로 찍히고 폰트는 그대로
\begin{verbatim}
\scriptsize
df = pd.read_csv(RAW_PATH)
\end{verbatim}

% 올바름 — \begin{verbatim} 앞에 스위치로
\scriptsize
\begin{verbatim}
df = pd.read_csv(RAW_PATH)
\end{verbatim}
```
**찾기** 전 파일을 한 번에 훑는다.
```bash
grep -rn -A1 'begin{verbatim}' week*/main.tex | grep -E '\\(tiny|scriptsize|footnotesize|small)$'
```
(같은 파일 위쪽의 「제목 뒤 `{...}` 가 부제로 먹힘」과는 **다른 버그**다. 그쪽은 그룹이 제목에
빨려들어가는 문제이고, 이쪽은 크기 명령이 verbatim 본문이 되는 문제다.)

## `verbatim` 앞에 크기 스위치를 **안 쓰면** 기본 크기로 조판돼 넘친다

바로 위 항목의 반대 실수다. 그쪽은 크기 명령이 verbatim **안**으로 들어간 경우이고, 이쪽은 **아예 없는** 경우다.
**증상** 컴파일 성공. `Overfull \vbox (14~30pt too high)`만 뜬다. 코드 블록만 유독 크게 보이지만 "원래 그런가 보다" 하고 넘기기 쉽다.
**원인** tcolorbox 안 `verbatim`은 바깥 `\scriptsize`를 상속하지 않는 자리(박스 본문을 새로 시작)가 많다. 4~6줄 코드 블록 하나가 `\normalsize`면 `\scriptsize` 대비 **0.6~1.0cm**를 더 먹는다 — 7cm 예산에서 치명적이다.
```latex
% 위험 -- 박스 본문이 기본 크기
\begin{tcolorbox}[conceptbox={입력}]
\begin{verbatim}
...
% 안전 -- \begin{verbatim} 바로 앞에 스위치
\begin{tcolorbox}[conceptbox={입력}, top=1pt, bottom=1pt]
\scriptsize
\begin{verbatim}
```
**찾기** 크기 스위치 없이 시작하는 verbatim 블록을 한 번에 센다.
```bash
grep -rn -B1 'begin{verbatim}' main.tex | grep -c 'tcolorbox'   # 후보 수
```
오버플로 프레임에 verbatim이 있으면 **제일 먼저 이것부터** 확인한다. 박스를 지우기 전에 스위치 한 줄로 해결되는 경우가 많다.
