# 한글 포함 pdflatex 로그를 grep이 binary로 취급 (검출 누락)

**증상** `grep "Overfull" main.log`가 아무것도 출력하지 않음(경고가 실제로는 존재). `grep -c`도 빈 출력 → "오버플로 0건"으로 오판하고 품질검사를 통과시킬 위험. `file main.log` = "Non-ISO extended-ASCII text".
**원인** kotex 한글 문서의 로그에 비UTF 바이트 시퀀스가 섞여 grep이 파일을 binary로 판정, 매치를 억제.
**해결** 로그 검사는 항상 `grep -a`(텍스트 강제): `grep -a -n "Overfull" main.log`. 에러 검색도 동일: `grep -a -A10 '^!' main.log`.

## `main.nav`의 널바이트가 진짜 에러를 100건의 가짜 에러로 덮는다

**증상** 3회 컴파일 후 `grep -a -c '^!'`가 **100건 이상**이고 전부
`! Text line contains an invalid character.` + `l.124 ...\slideentry {4}{0}{11}{50/50}{^^@^^@...`.
`Output written` 줄이 아예 없다(PDF 미생성).
**원인** kotex 문서의 `main.nav`에는 원래 비UTF 바이트가 섞인다(정상 문서도 `grep -c $'\0' main.nav`가 200 이상). 그런데 **1회차 컴파일에 진짜 에러가 하나라도 있으면** 그 상태로 쓰인 `.nav`를 2·3회차가 읽으며 널바이트마다 에러를 낸다. 그 소음이 원인을 완전히 가린다.
**해결** 보조파일을 지우고 **1회만** 돌려 첫 에러를 본다. 거기 나온 단 하나가 진짜다.
```bash
rm -f main.aux main.log main.nav main.out main.snm main.toc
pdflatex -interaction=nonstopmode main.tex >/dev/null 2>&1
grep -a -A6 '^!' main.log | head -20      # 이 첫 에러만 고치면 100건이 함께 사라진다
```
**판별법** `.nav` 널바이트 자체는 정상이다. 에러 개수가 갑자기 세 자리면 `.nav`를 의심하지 말고 1회차 로그를 본다.
