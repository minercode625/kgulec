# 고정 빌드 폴더를 두 세션이 동시에 쓰면 결과물이 섞인다

**증상** 컴파일은 성공하는데 `main.toc`가 **다른 주차 문서의 목차**이고, 페이지 수가 예상과 다르며,
`pdfinfo`가 `Syntax Error: Couldn't find trailer dictionary / Couldn't read xref table`를 낸다.
`ls -la $D`의 `main.tex` 크기가 방금 복사한 것과 다르다.
**원인** 번들 가이드의 격리 빌드 경로 `~/.claude/kgulec/tmp/build`는 **고정 경로**다.
같은 머신에서 kgulec 세션을 두 개 이상 돌리면(다른 주차를 동시에 빌드) 서로의 `main.tex`·`.aux`·`.pdf`를
덮어쓴다. `rm -rf $D`도 상대 세션이 곧바로 다시 만든다.
**해결** 빌드 폴더에 **대상 이름을 붙인다**.
```bash
D=~/.claude/kgulec/tmp/build-week6      # build 가 아니라 build-<대상>
rm -rf $D; mkdir -p $D
```
**확인** 컴파일 후 `grep -a 'Output written' main.log`의 페이지 수와 `main.toc`의 섹션명이
내 문서와 일치하는지 본다. 다르면 폴더가 오염된 것이다.
의심되면 `ps aux | grep pdflatex`로 다른 세션이 도는지 확인한다.

## 같은 원인의 변형: 두 세션이 같은 **주차 폴더**를 만든다

**증상** `weekN/figs/`에 내가 받지 않은 파일이 수십 개 있고, 내 `dl.py`가 방금 쓴
`figs/CREDITS.json`이 상대 세션의 기록을 통째로 덮어썼다(그 반대도 성립).
파일명 규칙이 두 가지로 섞여 있으면(`enc_sa_block.png` vs `enc_selfattn.png`) 확실하다.
**원인** 사용자가 여러 터미널에서 kgulec 세션을 동시에 돌리며 각각 다른 주차를 맡겼는데,
어느 한 세션이 **주차 번호를 잘못 골라** 이미 작업 중인 폴더에 들어갔다.
빌드 폴더와 달리 `weekN/`은 최종 산출물이라 덮어쓰면 복구가 어렵다.
**해결** 새 주차 작업을 시작하기 전에 **폴더가 이미 있는지, 최근에 수정됐는지** 본다.
```bash
ls -la "lecture note/"                       # weekN 폴더 목록 + mtime
find . -newermt "-15 minutes" -type f | head # 다른 세션이 방금 쓴 파일
ps aux | grep -c "[c]laude"                  # 동시 세션 수
```
이미 남이 쓰고 있으면 **그 폴더에 아무것도 쓰지 않는다.** 강의계획서로 내 주차를 다시 확인한다.
**사고 후 복구** 덮어쓴 `CREDITS.json`은 sha1 역조회로 일부 되살릴 수 있다
(Commons `action=query&list=allimages&aisha1=<sha1>`). 단 썸네일로 받은 파일은
원본과 sha1이 달라 매칭되지 않으므로, 상대 세션에 재실행을 알리는 편이 확실하다.

## 빌드 폴더가 아니라 **산출물 폴더**(weekN/)를 두 세션이 동시에 잡는다

**증상** 방금 내가 쓴 `tools/dl.py`가 **다른 파일명 규칙으로 바뀌어** 있다.
`figs/CREDITS.json`의 항목 수가 내가 만든 것과 다르고, `figs/`에 같은 그림이
서로 다른 이름으로 중복된다(`cnn_rnn_sa.png` / `cnn_rnn_selfattn.png`).
**원인** 격리 빌드 폴더뿐 아니라 **산출물 폴더 자체가 공유 자원**이다.
같은 주차를 두 세션에 지시하면 서로의 스크립트·메타데이터를 덮어쓴다.
**확인** `ps aux | grep '[c]laude'` 로 세션 수를, 폴더 mtime 으로 최근 쓰기를 본다.
**해결** 충돌이 확인되면 `~/.claude/kgulec/tmp/<주차>-mine/` 에 **전체 트리를 격리**해
만들고, 마지막에 설치한다. 설치 시 상대 파일은 지우지 말고 `.bak` 으로 보존한다.
```bash
cp "$D/tools/dl.py" "$D/tools/dl.py.other-session-backup"   # 덮어쓰기 전 보존
```
