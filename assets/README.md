# assets/

강의자료에서 공통으로 쓰는 고정 시각자료(로고 등)를 보관한다.
(특정 강의 전용 그림은 작업 디렉토리의 `./figs/`를 사용한다 — 여기에 두지 않는다.)

## 경기대 로고

- 파일명: **`kyonggi_logo.png`** (이름 고정 — `shared/beamer-templates.md`의 `\titlegraphic`이 이 이름을 찾는다)
- 형식: PNG 권장 (pdflatex는 png/pdf/jpg 지원, svg 불가)
- 용도: 타이틀 페이지(`\titlepage`) 로고

## 사용 방식 (자동)

타이틀 매크로:
```latex
\titlegraphic{\IfFileExists{kyonggi_logo.png}{\includegraphics[height=2.4cm]{kyonggi_logo.png}}{}}
```
`\IfFileExists`로 감싸여 있어, 로고가 없어도 컴파일은 깨지지 않는다(로고만 비표시).

`\titlegraphic`은 `main.tex`와 **같은 디렉토리**에서 파일을 찾으므로, 슬라이드 생성·수정 시 이 폴더의 `kyonggi_logo.png`를 출력(컴파일) 디렉토리로 복사해야 로고가 표시된다 (모드 A/B 절차에 포함됨).
