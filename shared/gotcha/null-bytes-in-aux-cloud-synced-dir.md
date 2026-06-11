# 클라우드 동기화 폴더에서 .aux/.pdf가 널바이트로 손상됨

## 증상

컴파일이 성공(`Output written on main.pdf (NN pages)`)했는데도 직후 `main.pdf`가 1~2KB로 잘려 있고 `pdftoppm`이 다음 에러를 낸다:

```
Syntax Error: Couldn't find trailer dictionary
Syntax Error: Couldn't read xref table
```

다시 `pdflatex` 하면 `.aux`를 읽다가 엉뚱한 에러:

```
! Extra }, or forgotten \endgroup.
l.265 }
! Text line contains an invalid character.
l.1 ...@^^@^^@^^@^^@^^@   (널바이트 ^^@)
```

`grep -c $'\0' main.aux` 가 0이 아니다(파일에 널바이트 섞임).

## 원인

작업 디렉토리가 iCloud/Dropbox/Google Drive 등 **클라우드 동기화 폴더** 안에 있으면, 동기화 데몬이 `pdflatex`가 쓰는 중인 `.aux`/`.pdf`를 가로채 부분 기록·널패딩하여 파일이 손상된다. 컴파일 자체는 정상이나 결과물이 사후에 깨진다. (경로에 한글/동기화 표식이 있는 데스크톱 폴더에서 특히)

## 해결 방법

**격리된 임시 디렉토리에서 컴파일하고 PDF만 복사해 온다.**

```bash
D=/tmp/beamerbuild; rm -rf $D; mkdir -p $D
cp main.tex *.png $D/ 2>/dev/null; cp -r figs $D/ 2>/dev/null
cd $D && pdflatex -interaction=nonstopmode -halt-on-error main.tex \
  && pdflatex -interaction=nonstopmode main.tex \
  && pdflatex -interaction=nonstopmode main.tex   # overlay/TOC 위해 다회
cp $D/main.pdf /원래/경로/main.pdf
```

손상된 보조파일은 먼저 지운다: `rm -f main.aux main.toc main.nav main.snm main.out main.vrb`.

## 일반 규칙

- `remember picture/overlay`(섹션 divider, 커스텀 cover)는 cross-ref 위해 최소 2회 컴파일 필요.
- PDF 렌더 검사(`pdftoppm`)는 컴파일과 **동시에 같은 파일을 읽지 않도록** 컴파일 완료 후 실행.
