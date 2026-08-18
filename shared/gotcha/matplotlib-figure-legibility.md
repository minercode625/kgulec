# matplotlib 그림의 라벨이 슬라이드에서 안 읽힌다 (잘린 폭 문제)

**증상** 컴파일 정상, Overfull 경고 없음. 그런데 PDF에서 그림 안 글자가 3pt 수준으로 뭉개져 읽히지 않는다. `plt.subplots(1, 4, figsize=(11.4, 3))` 같은 넓은 다중 패널에서 특히 심하다. **시각 검사로만 발견된다.**
**원인** 슬라이드 위 글자 크기는 `fontsize_pt / (저장된 그림의 인치 폭)`에 비례한다. `\includegraphics[width=0.95\textwidth]`는 그림을 항상 약 11.5cm에 눌러 넣으므로, 11인치(28cm) 폭 그림은 배율 0.4 → 9pt 라벨이 3.6pt가 된다. `savefig(bbox_inches="tight")` + `ax.set_aspect("equal")`이면 **figsize가 아니라 데이터 종횡비**가 최종 크기를 정해 예측이 더 어긋난다.
**해결** 저장 직후 실제 크기를 인치로 찍어 **6~9인치 밴드**를 유지한다. 넘으면 figsize를 줄이거나 패널 수를 줄인다(4개 -> 3개). 폰트를 키우는 것보다 그림을 좁히는 편이 확실하다.
```python
def save(fig, name):
    fig.savefig(OUT + name)                       # rcParams: figure.dpi=160
    import PIL.Image
    w, h = PIL.Image.open(OUT + name).size
    print(f"{name} {w/160:.1f} x {h/160:.1f} in" + ("  <-- TOO WIDE" if w/160 > 9.4 else ""))
```
**일반 규칙** 데이터 종횡비 목표는 2.5~3.5:1이다(슬라이드 11.5cm 폭 x 약 4cm 높이). aspect equal 그림은 xlim/ylim 범위비가 곧 최종 종횡비이므로, 설명 문장은 그림 밖(슬라이드 tcolorbox)으로 빼서 세로 범위를 줄인다.

## 이미지 타일을 격자로 붙일 때 여백이 없으면 세로 줄무늬로 보인다

**증상** 작은 이미지($8\times8$ 등) 수백 개를 `np.block` 으로 이어 붙이면
슬라이드에서 개별 이미지가 아니라 **세로 줄무늬 벽지**로 읽힌다. 에러·경고 없음.
**원인** 타일 사이에 경계가 없어 이웃 타일의 밝은 열과 어두운 열이 한 줄로 융합된다.
**해결** 타일마다 1~2px 흰 여백(gutter)을 넣고 `interpolation="nearest"` 로 그린다.
```python
pad, tile = 1, 8 + 2 * 1
canvas = np.zeros((n * tile, n * tile))
for r in range(n):
    for c in range(n):
        canvas[r*tile+pad:r*tile+pad+8, c*tile+pad:c*tile+pad+8] = im[r, c]
```
