# verbatim 안의 탭 문자가 XeLaTeX에서 이상한 글리프로 찍힌다

**증상** 컴파일 에러·Overfull 경고 없음. 그런데 코드 블록 들여쓰기 자리에
백틱 비슷한 기호(`` ` ``)가 줄마다 찍히고 정렬이 어긋난다. **PDF 렌더로만 발견**된다.
(예: YAML 예시의 `branches:` 아래 `- master` 줄들이 `` ` - master `` 처럼 보인다.)

**원인** `verbatim` 은 탭(U+0009)을 공백으로 바꾸지 않는다. XeLaTeX + 타자기체에서
탭이 폰트의 다른 글리프 슬롯으로 매핑돼 눈에 보이는 문자로 조판된다.
에디터에서 탭으로 들여쓴 기존 .tex를 그대로 옮길 때 잘 걸린다.

**해결** verbatim 블록 안의 탭을 공백으로 치환한다(내용은 그대로 유지).

```bash
grep -n -P '\t' main.tex        # 탭 있는 줄 찾기
```
```python
lines=open("main.tex",encoding="utf-8").read().split("\n")
out=[]; inverb=False
for l in lines:
    if l.strip().startswith("\\begin{verbatim}"): inverb=True
    elif l.strip().startswith("\\end{verbatim}"): inverb=False
    out.append(l.replace("\t","  ") if inverb else l)
open("main.tex","w",encoding="utf-8").write("\n".join(out))
```

**일반 규칙** 코드 블록은 탭 없이 공백 들여쓰기로만 작성한다.
관련: `beamer-frame-pitfalls.md`(크기 명령이 verbatim 안으로 들어가는 별개 버그).
