# Beamer Templates Reference

이 문서는 LaTeX Beamer 슬라이드 생성 시 사용할 표준 프리앰블과 슬라이드 패턴 모음이다.

---

## 1. 표준 프리앰블

아래 프리앰블을 **그대로 복사**하여 사용한다. `\course`, `\week`, `\topic`, `\professor`, `\department`, `\naljja` 값만 강의 내용에 맞게 변경한다(이 중 `\professor`·`\department` 및 PRIMARY THEME 4색은 보통 사용자 config에서 자동 채워짐 — 모드 A가 처리). `% === PRIMARY THEME ===` 블록의 4색은 `/kgulec:setup` 가 교체하는 영역이다. 그 외 부분은 수정하지 않는다.

```latex
\documentclass[xcolor=svgnames,table]{beamer}
\usepackage[utf8]{inputenc}
\usepackage{xcolor}
\usepackage{booktabs, comment}
\usepackage{pgfpages}
\usepackage{csquotes}
\usepackage{amsmath}
\usepackage{tikz}
\usetikzlibrary{shapes, arrows.meta, positioning, shadows, calc, decorations.pathreplacing, backgrounds, fit, matrix}
\usepackage{multicol}
\usepackage{multirow}
\usepackage{kotex}
\usepackage{bbm}
\usepackage{graphicx}
\usepackage{algorithm2e}
\usepackage{hyperref}
\usepackage{verbatim}
\usepackage{array}
\usepackage{fp}
\usepackage[most]{tcolorbox}
\usepackage{fontawesome5}
\usepackage{pifont}
\usepackage{ragged2e}

\RestyleAlgo{ruled}
\SetKwInput{KwIn}{Input}
\SetKwInput{KwOut}{Output}
\usetheme{Madrid}

% === PRIMARY THEME (주색 4색 램프 — /kgulec:setup 가 교체) ===
% 프리셋 적용 시 정확히 이 4줄만 교체된다. 의미색(example/warning/info)은 아래에서 고정.
\definecolor{mqblue}{RGB}{91, 155, 213}       % base  (주색)
\definecolor{mqdeepblue}{RGB}{47, 85, 151}    % deep
\definecolor{mqgrayblue}{RGB}{79, 129, 189}   % mid
\definecolor{mqlightblue}{RGB}{221, 235, 247} % light
% === END PRIMARY THEME ===

% === COLORS (의미색 고정) ===
\definecolor{mqmagenta}{RGB}{198, 0, 126}
\colorlet{conceptbg}{mqlightblue}   % conceptbox 배경 = 주색 light (주색 자동 추종)
\colorlet{conceptfr}{mqdeepblue}    % conceptbox 테두리 = 주색 deep (주색 자동 추종)
\definecolor{examplebg}{RGB}{226, 240, 217}
\definecolor{examplefr}{RGB}{56, 118, 29}
\definecolor{warningbg}{RGB}{255, 235, 204}
\definecolor{warningfr}{RGB}{191, 100, 0}
\definecolor{infobg}{RGB}{237, 237, 237}
\definecolor{infofr}{RGB}{100, 100, 100}
\definecolor{accentred}{RGB}{192, 57, 43}
\definecolor{accentpurple}{RGB}{142, 68, 173}
\definecolor{accentteal}{RGB}{22, 160, 133}
\definecolor{lightcream}{RGB}{252, 249, 242}
\definecolor{darktext}{RGB}{45, 45, 45}

\usecolortheme[named=mqblue]{structure}
\setbeamercolor{normal text}{fg=darktext}
\setbeamercolor{title in head/foot}{bg=mqlightblue, fg=mqgrayblue}
\setbeamercolor{author in head/foot}{bg=mqdeepblue, fg=white}
\setbeamercolor{page number in head/foot}{bg=mqdeepblue, fg=mqlightblue}
\setbeamercolor{frametitle}{fg=mqdeepblue}
\setbeamerfont{frametitle}{series=\bfseries,size=\large}
\setbeamerfont{title}{series=\bfseries}
\setbeamerfont{block title}{series=\bfseries}
\setbeamertemplate{navigation symbols}{}
\setbeamersize{text margin left=0.45cm, text margin right=0.45cm}
\setlength{\leftmargini}{1.1em}
\renewcommand{\arraystretch}{1.18}

\DeclareMathOperator*{\argmax}{arg\,max}
\DeclareMathOperator*{\argmin}{arg\,min}

% === BOX STYLES ===
\tcbset{
  conceptbox/.style={
    colback=conceptbg, colframe=conceptfr,
    boxrule=0.8pt, arc=3pt,
    fonttitle=\bfseries,
    left=6pt, right=6pt, top=4pt, bottom=4pt,
    title=#1
  },
  examplebox/.style={
    colback=examplebg, colframe=examplefr,
    boxrule=0.8pt, arc=3pt,
    fonttitle=\bfseries,
    left=6pt, right=6pt, top=4pt, bottom=4pt,
    title=#1
  },
  warningbox/.style={
    colback=warningbg, colframe=warningfr,
    boxrule=0.8pt, arc=3pt,
    fonttitle=\bfseries,
    left=6pt, right=6pt, top=4pt, bottom=4pt,
    title=#1
  },
  infobox/.style={
    colback=infobg, colframe=infofr,
    boxrule=0.8pt, arc=3pt,
    fonttitle=\bfseries,
    left=6pt, right=6pt, top=4pt, bottom=4pt,
    title=#1
  },
  mathbox/.style={
    colback=conceptbg!35, colframe=conceptfr,
    boxrule=1pt, arc=4pt,
    left=8pt, right=8pt, top=6pt, bottom=6pt
  },
  taskcard/.style={
    colback=white, colframe=#1,
    boxrule=1.15pt, arc=4pt,
    left=5pt, right=5pt, top=3pt, bottom=3pt,
    shadow={1pt}{-1pt}{0pt}{black!20}
  }
}

% === CUSTOM COMMANDS ===
\newcommand{\numcircle}[1]{%
  \tikz[baseline=(char.base)]{
    \node[shape=circle, draw=mqblue, fill=mqblue, text=white, inner sep=1.5pt, font=\bfseries\small] (char) {#1};
  }%
}

\newcommand{\sectiondividerframe}[3]{%
\begin{frame}[plain]
\begin{tikzpicture}[remember picture,overlay]
  \fill[mqdeepblue] (current page.north west) rectangle (current page.south east);
  \fill[mqlightblue] ([yshift=-1.25cm]current page.north west) rectangle ([yshift=-2.35cm]current page.north east);
  \fill[white] ([xshift=0.85cm,yshift=-2.85cm]current page.north west) rectangle ([xshift=-0.85cm,yshift=0.9cm]current page.south east);
  \fill[conceptbg] ([xshift=0.85cm,yshift=-2.85cm]current page.north west) rectangle ([xshift=1.15cm,yshift=0.9cm]current page.south east);
\end{tikzpicture}
\vspace{1.25cm}
\begin{center}
  {\color{mqblue}\fontsize{42}{42}\selectfont #3}\\[0.35cm]
  {\Huge\bfseries #1}\\[0.22cm]
  {\large\color{mqdeepblue} #2}
\end{center}
\vfill
\end{frame}
}

\newcommand{\iffileincludegraphics}[3]{%
  \IfFileExists{#1}{\includegraphics[#2]{#1}}{#3}%
}

% === FOOTLINE ===
\makeatletter
\setbeamertemplate{footline}{
  \leavevmode%
  \hbox{%
  \begin{beamercolorbox}[wd=.5\paperwidth,ht=2.25ex,dp=1ex,center]{author in head/foot}%
    \usebeamerfont{author in head/foot}\insertshortauthor\expandafter\ifblank\expandafter{\beamer@shortinstitute}{}{~~(\insertshortinstitute)}
  \end{beamercolorbox}%
  \begin{beamercolorbox}[wd=.4\paperwidth,ht=2.25ex,dp=1ex,center]{title in head/foot}%
    \usebeamerfont{title in head/foot}\insertshorttitle
  \end{beamercolorbox}%
  \begin{beamercolorbox}[wd=.1\paperwidth,ht=2.25ex,dp=1ex,center]{page number in head/foot}%
    \usebeamerfont{page number in head/foot}\insertframenumber{} / \inserttotalframenumber
  \end{beamercolorbox}}%
  \vskip0pt%
}
\makeatother

% === TITLE META (CHANGE THESE) ===
% \professor·\department 는 보통 /kgulec:setup 의 사용자 설정(config)에서 자동 채워진다.
\newcommand{\course}{과목명}
\newcommand{\week}{1}
\newcommand{\topic}{Topic Title}
\newcommand{\professor}{교수명}
\newcommand{\department}{소속}
\newcommand{\naljja}{26.03.18}

\FPeval\result{\week+1}
\FPround\nextweek{\result}{0}

\title[\week 주차 \topic]{\course}
\author[\course]{\topic \\[6pt] {\small \professor}}
\institute[]{\department \quad \week 주차}
\date{\naljja}
\titlegraphic{\IfFileExists{kyonggi_logo.png}{\includegraphics[height=2.4cm]{kyonggi_logo.png}}{}}

\begin{document}

\begin{frame}
  \titlepage
\end{frame}

\begin{frame}{목차}
  \tableofcontents
\end{frame}

% === CONTENT STARTS HERE ===

\end{document}
```

---

## 2. 슬라이드 패턴

### 2.1 개념 설명 슬라이드 (2열 + 하단 정보)

```latex
\begin{frame}{Bayes' Theorem}
  \begin{tcolorbox}[conceptbox={\faIcon{balance-scale} Main Idea}]
    \small Bayes' Theorem updates a belief after we observe new evidence.
  \end{tcolorbox}
  \vspace{0.06cm}
  \begin{tcolorbox}[mathbox]
    \centering
    \large
    $P(A \mid B) = \dfrac{P(B \mid A)\,P(A)}{P(B)}, \qquad P(B)>0$
  \end{tcolorbox}
  \vspace{0.08cm}
  \begin{tcolorbox}[examplebox={\faIcon{stethoscope} Example}]
    \scriptsize Positive test $\Rightarrow$ updated probability of disease.
  \end{tcolorbox}
\end{frame}
```

### 2.2 비교 슬라이드 (2열)

```latex
\begin{frame}{Prior vs. Evidence}
  \textbf{Prior belief and evidence are the two ingredients of Bayesian thinking.}
  \vspace{0.16cm}
  \begin{columns}[T,onlytextwidth]
    \column{0.49\textwidth}
      \begin{tcolorbox}[conceptbox={\faIcon{cloud} Prior Belief}]
        \small What we assume \textbf{before} seeing new data.
        \[
          P(A)
        \]
      \end{tcolorbox}
    \column{0.49\textwidth}
      \begin{tcolorbox}[examplebox={\faIcon{search} Evidence}]
        \small New information that helps us \textbf{update} the belief.
        \[
          B \quad \Rightarrow \quad P(A\mid B)
        \]
      \end{tcolorbox}
  \end{columns}
  \vspace{0.12cm}
  \begin{tcolorbox}[infobox={\faIcon{lightbulb} Intuition}]
    \small \textbf{Posterior} means: belief \textbf{after} looking at evidence.
  \end{tcolorbox}
\end{frame}
```

### 2.3 3열 아이콘 카드 (개요/분류)

```latex
\begin{frame}{Why This Matters in NLP}
  \textbf{Entropy and cross-entropy appear all over modern NLP.}
  \vspace{0.16cm}
  \begin{columns}[T,onlytextwidth]
    \column{0.32\textwidth}
      \begin{tcolorbox}[taskcard=mqblue, halign=center, equal height group=nlpuse]
        {\color{mqblue}\Large\faIcon{envelope}}\\[2pt]
        {\scriptsize\textbf{Text Classification}}\\
        {\tiny cross-entropy loss}
      \end{tcolorbox}
    \column{0.32\textwidth}
      \begin{tcolorbox}[taskcard=accentpurple, halign=center, equal height group=nlpuse]
        {\color{accentpurple}\Large\faIcon{language}}\\[2pt]
        {\scriptsize\textbf{Language Modeling}}\\
        {\tiny better probabilities = lower loss}
      \end{tcolorbox}
    \column{0.32\textwidth}
      \begin{tcolorbox}[taskcard=examplefr, halign=center, equal height group=nlpuse]
        {\color{examplefr}\Large\faIcon{chart-bar}}\\[2pt]
        {\scriptsize\textbf{Evaluation}}\\
        {\tiny compare models by surprise}
      \end{tcolorbox}
  \end{columns}
  \vspace{0.12cm}
  \begin{tcolorbox}[conceptbox={\faIcon{check-circle} Bottom Line}]
    \small Key takeaway sentence here.
  \end{tcolorbox}
\end{frame}
```

### 2.4 수식 슬라이드

```latex
\begin{frame}{Maximum Likelihood Estimation}
  \begin{tcolorbox}[conceptbox={\faIcon{search} What does MLE do?}]
    \small \textbf{MLE} finds the parameter $\theta$ that makes the observed data most likely.
  \end{tcolorbox}
  \vspace{0.12cm}
  \begin{tcolorbox}[mathbox]
    \centering
    \[
      P(X\mid \theta)=\prod_{i=1}^{n}P(x_i\mid \theta)
      \qquad \text{(if samples are i.i.d.)}
    \]
    \[
      \hat{\theta}=\argmax_{\theta}P(X\mid \theta)
    \]
  \end{tcolorbox}
  \vspace{0.08cm}
  \begin{tcolorbox}[examplebox={\faIcon{coins} Example}]
    \small Flip a coin 10 times, observe 7 heads and 3 tails.
  \end{tcolorbox}
\end{frame}
```

### 2.5 TikZ 다이어그램 + 설명

```latex
\begin{frame}{Understanding Bayes in AI}
  \begin{tcolorbox}[conceptbox={\faIcon{robot} Classification View}]
    \small Bayes' Theorem helps decide which class is more likely.
  \end{tcolorbox}
  \vspace{0.15cm}
  \begin{center}
  \begin{tikzpicture}[>=Stealth, node distance=1.15cm,
    stepbox/.style={draw, rounded corners=4pt, minimum width=2.85cm,
      minimum height=1.05cm, align=center, font=\footnotesize, thick,
      fill=white, drop shadow={shadow xshift=1pt, shadow yshift=-1pt, opacity=0.25}}
  ]
    \node[stepbox, draw=mqblue, fill=conceptbg] (in) {\textbf{Input}\\observed data};
    \node[stepbox, draw=accentpurple, fill=accentpurple!10, right=of in] (proc) {\textbf{Process}\\clue extraction};
    \node[stepbox, draw=examplefr, fill=examplebg, right=of proc] (out) {\textbf{Output}\\prediction};
    \draw[->, thick, mqblue] (in) -- (proc);
    \draw[->, thick, examplefr] (proc) -- (out);
  \end{tikzpicture}
  \end{center}
\end{frame}
```

### 2.6 표 슬라이드

```latex
\begin{frame}{Algorithm Comparison}
  \textbf{All three use a priority queue, but prioritize differently.}
  \vspace{0.16cm}
  \begin{center}
  \footnotesize
  \begin{tabular}{>{\centering\arraybackslash}m{2.2cm}|>{\centering\arraybackslash}m{2.3cm}|>{\arraybackslash}m{4.8cm}}
    \textbf{Algorithm} & \textbf{Priority} & \textbf{Interpretation} \\ \hline
    UCS & $g(n)$ & Cheapest path found so far \\ \hline
    Greedy & $h(n)$ & Looks closest to the goal \\ \hline
    A* & $g(n)+h(n)$ & Best total cost estimate
  \end{tabular}
  \end{center}
  \vspace{0.18cm}
  \begin{tcolorbox}[infobox={\footnotesize \faIcon{balance-scale} Takeaway}]
    \footnotesize
    Brief summary of the comparison.
  \end{tcolorbox}
\end{frame}
```

### 2.7 TikZ 형태소 분석 등 (한글 예시)

```latex
\begin{frame}{형태소 예시}
  \textbf{문장 분석: \textit{나는 컴퓨터 공부가 좋아.}}
  \vspace{0.2cm}
  \begin{center}
  \begin{tikzpicture}[node distance=0.08cm,
    lex/.style={draw=examplefr, fill=examplebg, rounded corners=3pt,
      minimum height=0.85cm, font=\small\bfseries, inner xsep=6pt},
    gram/.style={draw=warningfr, fill=warningbg, rounded corners=3pt,
      minimum height=0.85cm, font=\small\bfseries, inner xsep=6pt}]
    \node[lex] (n1) {나};
    \node[gram, right=of n1] (n2) {는};
    \node[lex, right=of n2] (n3) {컴퓨터};
    \node[lex, right=of n3] (n4) {공부};
    \node[gram, right=of n4] (n5) {가};
    \node[lex, right=of n5] (n6) {좋-};
    \node[gram, right=of n6] (n7) {아};
  \end{tikzpicture}
  \end{center}
  \vspace{0.18cm}
  \begin{columns}[T,onlytextwidth]
    \column{0.48\textwidth}
      \begin{tcolorbox}[examplebox={\faIcon{check-circle} 실질형태소}]
        \scriptsize 나, 컴퓨터, 공부, 좋-
      \end{tcolorbox}
    \column{0.48\textwidth}
      \begin{tcolorbox}[warningbox={\faIcon{paperclip} 형식형태소}]
        \scriptsize 는, 가, 아
      \end{tcolorbox}
  \end{columns}
\end{frame}
```

### 2.8 섹션 구분 슬라이드

```latex
\section{Section Name}
\sectiondividerframe{Main Title}{Subtitle description}{\faIcon{icon-name}}
```

### 2.9 요약 / Next Class 슬라이드

```latex
\begin{frame}{Week Summary}
  \textbf{Today's Key Takeaways}
  \vspace{0.14cm}
  \begin{enumerate}
  \footnotesize
  \item First key point.
  \vspace{0.10cm}
  \item Second key point.
  \vspace{0.10cm}
  \item Third key point.
  \end{enumerate}
  \vspace{0.16cm}
  \begin{center}
  \footnotesize
  \textbf{Big picture:} One-sentence summary.
  \end{center}
\end{frame}

\begin{frame}{Next Class}
  \textbf{Week \nextweek: Next Topic Title}
  \vspace{0.28cm}
  \begin{columns}[T,onlytextwidth]
    \column{0.32\textwidth}
      \begin{tcolorbox}[taskcard=mqblue, halign=center]
        {\color{mqblue}\Large\faIcon{icon1}}\\[2pt]
        {\footnotesize\textbf{Sub-topic 1}}\\
        {\scriptsize brief description}
      \end{tcolorbox}
    \column{0.32\textwidth}
      \begin{tcolorbox}[taskcard=accentpurple, halign=center]
        {\color{accentpurple}\Large\faIcon{icon2}}\\[2pt]
        {\footnotesize\textbf{Sub-topic 2}}\\
        {\scriptsize brief description}
      \end{tcolorbox}
    \column{0.32\textwidth}
      \begin{tcolorbox}[taskcard=examplefr, halign=center]
        {\color{examplefr}\Large\faIcon{icon3}}\\[2pt]
        {\footnotesize\textbf{Sub-topic 3}}\\
        {\scriptsize brief description}
      \end{tcolorbox}
  \end{columns}
\end{frame}
```

---

## 3. 자주 쓰는 색상 조합 (taskcard용)

| 용도 | 색상 이름 |
|------|-----------|
| 기본/메인 | `mqblue` |
| 보조/대비 | `accentpurple` |
| 긍정/성공 | `examplefr` |
| 경고/주의 | `warningfr` |
| 강조/위험 | `accentred` |
| 청록/보조 | `accentteal` |

---

## 4. 이미지 처리

이미지가 필요한 위치에는 실제 `\includegraphics`를 넣지 않고 주석을 남긴다:

```latex
% TODO: 여기에 [신경망 구조 다이어그램] 이미지 삽입
% \includegraphics[width=0.8\textwidth]{filename.pdf}
```

대학 로고만 예외로 `\IfFileExists`를 사용한다 (프리앰블에 이미 포함).
