# 시각검사용 `magick montage`가 원본 그림을 덮어씀

**증상** 그림 컨택트시트를 만든 뒤 `figs/`의 그림 일부가 **다른 그림들이 격자로 붙은 시트 이미지**로 바뀐다. 컴파일은 되지만 슬라이드에 엉뚱한 그림이 박힌다. 에러 없음.
**원인** `montage`는 **마지막 인수를 출력 파일**로 본다. `xargs`로 파일 목록을 뒤에 붙이면 출력 경로가 중간으로 밀리고 **목록의 마지막 파일이 출력 대상**이 되어 덮어써진다.
```bash
# 위험: 출력이 중간 -> 마지막 입력 파일이 덮어써진다
ls gen_*.png | xargs magick montage -tile 4x4 -geometry 400x300 sheet.png
# 안전: 파일 목록 먼저, 출력 경로는 항상 맨 끝
magick montage gen_*.png -tile 4x4 -geometry 400x300+3+3 -background white sheet.png
```
**해결** 출력 경로를 **명령의 맨 끝**에 두고 `xargs`를 쓰지 않는다. 목록을 나눠 여러 시트로 만들 때는 셸 배열 슬라이스를 쓴다(zsh 배열은 1-기반이라 `${A[@]:0:15}` 같은 슬라이스 결과가 bash와 다를 수 있으니 결과를 확인한다). 사고가 나면 `mkfigs.py`를 다시 돌려 복구한다.

## zsh 에서 파일 목록을 변수에 담아 넘기면 한 덩어리로 전달된다

**증상** `montage: unable to open image ' p-001.png p-002.png ...': No such file or directory`
이어서 `missing an image filename 'sheet0.png'`. 시트가 하나도 안 만들어진다.
**원인** zsh 는 **따옴표 없는 변수도 단어 분리하지 않는다**(bash 와 다르다).
`files="$files $f"` 로 쌓아 `magick montage $files ...` 로 넘기면 목록 전체가 파일명 하나가 된다.
**해결** 셸 배열을 쓰거나(`files+=("$f")` → `magick montage $files ...`), 목록 생성을 파이썬에 맡긴다.
```python
import glob, subprocess
fs = sorted(glob.glob("p-*.png"))
for i in range(0, len(fs), 16):
    subprocess.run(["magick","montage",*fs[i:i+16],"-tile","4x4",
                    "-geometry","390x293+3+3", f"sheet{i//16}.png"], check=True)
```
페이지가 100장을 넘으면 어차피 여러 시트로 쪼개야 하므로 파이썬 쪽이 편하다.

## `pdftoppm` 출력 파일명의 자릿수는 **전체 페이지 수**를 따른다

**증상** `pdftoppm -f 93 -l 93 main.pdf pg93` 후 `magick montage pg93-093.png ...` 가
`unable to open image 'pg93-093.png': No such file or directory`.
**원인** 접미 번호의 자릿수는 요청 페이지가 아니라 **문서 전체 페이지 수**로 정해진다.
95쪽 문서면 두 자리(`pg93-93.png`), 100쪽 이상이면 세 자리(`pg93-093.png`).
**해결** 자릿수를 추측하지 말고 글롭으로 받는다: `magick montage pg93*.png ...`.
