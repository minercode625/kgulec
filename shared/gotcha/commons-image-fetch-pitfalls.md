# Wikimedia Commons 그림 내려받기 함정 (pdflatex 가 죽는다)

`dl.py` 류로 Commons 이미지를 받아 `figs/` 에 넣을 때 반복해서 걸리는 두 가지.

## 실제로는 GIF 인 파일을 `.png` 로 저장 → `libpng: internal error`

**증상** `!pdfTeX error: pdflatex (file ./figs/x.png): libpng: internal error` → `Fatal error occurred, no output PDF file produced!`. 컴파일이 그 지점에서 **완전히 중단**되므로 뒤쪽 페이지의 오버플로 검사도 함께 누락된다.
**원인** GIF 원본은 썸네일 렌더가 없어 API 가 **원본 GIF URL** 을 그대로 돌려준다. 확장자만 `.png` 로 붙여 저장하면 내용은 GIF다. pdflatex 는 확장자로 디코더를 고른다.
**해결** 내려받은 뒤 매직바이트를 검사해 재인코딩한다.
```python
with open(dest, "rb") as f:
    magic = f.read(8)
if ext == ".png" and magic[:4] != b"\x89PNG":
    os.system(f'magick "{dest}" -strip -define png:color-type=2 "{dest}.tmp.png" && mv "{dest}.tmp.png" "{dest}"')
```

## thumburl 의 `?utm_*` 쿼리 때문에 확장자가 쓰레기값이 됨

**증상** `figs/` 에 `gradient_field.org&utm_campaign=imageinfo&utm_content=original` 같은 파일이 생기고 `\includegraphics` 가 못 찾는다.
**원인** Commons API 의 `thumburl` 에 추적 쿼리스트링이 붙는다. `os.path.splitext(url)` 이 쿼리 끝부분을 확장자로 집는다.
**해결** 확장자 판정 전에 쿼리를 제거한다.
```python
def path_of(url):
    return urllib.parse.urlsplit(url).path      # ?utm_* 제거
ext = os.path.splitext(path_of(url))[1].lower()
```

## 일반 규칙
SVG·GIF 는 pdflatex 가 못 읽는다 → 반드시 래스터 썸네일(`iiurlwidth`)로 받고, 받은 뒤 `magick identify` 로 실제 포맷을 확인한다.
