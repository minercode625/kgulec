# 노트북 생성 스크립트(mknb.py): r-string 셀 안에 `"""` 를 못 넣는다

**증상** 생성된 `.ipynb` 셀이 실행 즉시 `SyntaxError: unexpected character after line continuation character`.
셀 소스를 보면 `\"\"\"설명\"\"\"` 이 그대로 들어 있다. 검증 스크립트에서 그 노트북만 FAIL.

**원인** 셀 본문을 `code(r"""...""")` 로 쓰면 r-string 안에서 `\"` 가 이스케이프되지 **않고**
백슬래시+따옴표 그대로 남는다. r-string 은 `"""` 자체를 포함할 수도 없다.
(r 을 쓰는 이유는 코드 안 `"\n"` 과 마크다운의 `\alpha` 를 보존하기 위해서이므로 r 은 버리면 안 된다.)

**해결** 셀 안 독스트링은 작은따옴표 3개로 쓴다.

```python
# 위험 -- 생성된 셀이 SyntaxError
code(r'''
def f(x):
    \"\"\"설명\"\"\"
''')

# 안전
code(r"""
def f(x):
    '''설명'''
""")
```

## `HTML(anim.to_jshtml())` 이 20MB 임베드 한도를 넘는다

**증상** 셀 실행 중 경고 `Animation size has reached 21194164 bytes, exceeding the limit of
20971520.0. ... This and further frames will be dropped.` 뒤쪽 프레임이 **조용히 잘린 채**
노트북이 저장된다(에러 아님). `.ipynb` 도 수십 MB가 되어 Colab 업로드가 느려진다.

**원인** `to_jshtml()` 은 프레임마다 PNG를 base64로 박아 넣는다.
**잡음이 낀 RGB 이미지는 PNG 압축이 거의 안 되므로** 프레임 하나가 수백 KB다.
확산 모델 노트북(노이즈 이미지 애니메이션)에서 특히 잘 터진다.

**해결** 프레임 수와 **렌더 픽셀 수**를 함께 줄인다. 원본 해상도를 낮추고
`interpolation="nearest"` 로 키워 보여 주면 같은 크기에서 PNG가 훨씬 잘 압축된다.
```python
# 위험: 120x120 RGB 잡음 x 60 프레임, figsize 3.6 -> 21MB 초과
# 안전: 72x72 로 낮추고 40 프레임, figsize 3.0
im = ax.imshow(img, interpolation="nearest")
```
산점도·회색조 격자 애니메이션은 잘 압축되므로 프레임을 100개까지 써도 된다.
