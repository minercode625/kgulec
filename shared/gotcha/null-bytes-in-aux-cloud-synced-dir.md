# 클라우드 동기화 폴더에서 .aux/.pdf가 널바이트로 손상

**증상** 컴파일 성공(`Output written ...`)했는데 `main.pdf`가 1~2KB로 잘림. `pdftoppm`이 `Syntax Error: Couldn't find trailer dictionary / read xref table`. 재컴파일 시 `.aux`에서 `Extra }`, `Text line contains an invalid character`(널바이트 `^^@`). `grep -c $'\0' main.aux` ≠ 0.
**원인** 작업 폴더가 iCloud/Dropbox/Google Drive 동기화 폴더면, 동기화 데몬이 쓰는 중인 `.aux`/`.pdf`를 가로채 부분기록·널패딩해 손상. 컴파일은 정상이나 결과물이 사후 깨짐.
**해결** 격리 임시 폴더에서 컴파일 후 PDF만 복사:
```bash
D=/tmp/beamerbuild; rm -rf $D; mkdir -p $D
cp main.tex *.png $D/ 2>/dev/null; cp -r figs $D/ 2>/dev/null
cd $D && pdflatex -interaction=nonstopmode -halt-on-error main.tex \
  && pdflatex -interaction=nonstopmode main.tex \
  && pdflatex -interaction=nonstopmode main.tex   # overlay/TOC 위해 다회
cp $D/main.pdf /원래/경로/main.pdf
```
손상 보조파일 먼저 삭제: `rm -f main.aux main.toc main.nav main.snm main.out main.vrb`.
**일반 규칙** overlay/remember picture는 cross-ref 위해 최소 2회 컴파일. PDF 렌더 검사는 컴파일 완료 후(동시에 같은 파일 읽지 않게).
