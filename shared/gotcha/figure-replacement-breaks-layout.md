# 기존 덱의 그림을 교체하면 「한 프레임 2장」 슬라이드가 조용히 넘친다

**증상** 모드 B에서 `figs/pracN.png` 를 새로 만든 스크린샷으로 갈아끼웠다. 텍스트는 한 글자도
안 건드렸는데 `Overfull \vbox (30~140pt too high)` 가 여러 주차에서 새로 생긴다.
PDF는 정상 생성되고, 넘친 슬라이드는 **아래쪽 그림이 footline에 잘린 채** 나온다.

**원인** 원본 스크린샷은 터미널을 좁게 크롭한 **아주 납작한** 이미지(종횡비 0.2~0.3)였는데,
새로 만든 이미지는 타이틀 바·전체 출력이 들어가 **더 높다**(0.35~0.6). 프레임의 `width=` 는
그대로이므로 높이만 늘어나 예산을 넘는다. 한 프레임에 그림이 2장이면 바로 터진다.

**높이 예산** 4:3 beamer 본문 높이 ≈ 7cm, `\textwidth` ≈ 10.8cm 이므로

    슬라이드 차지 높이 ≈ width계수 × 10.8cm × (이미지 세로/가로)

- 그림 **1장** + 짧은 텍스트: `width계수 × 종횡비 ≤ 0.60`
- 그림 **2장** + 각 단계 설명: 장당 `width계수 × 종횡비 ≤ 0.24`

**해결** 교체 후 반드시 재계산한다. 셋 중 하나.
1. `width=` 를 줄인다(2장짜리는 보통 0.66 이하).
2. 예산을 못 맞추면 **프레임을 쪼갠다**(Step 3 / Step 4 를 각각 한 장씩).
3. 애초에 이미지를 납작하게 만든다(출력 줄 수를 줄이고 타이틀 바를 뺀다).

**검사** 로그의 Overfull 만 보지 말고 렌더해서 눈으로 확인한다. 어두운 스크린샷은
footline 바로 위 띠에 잉크가 많아 자동 검출이 **오탐**을 잘 낸다.
```bash
python3 - <<'PY'
import re, pathlib
for f in sorted(pathlib.Path(".").glob("week*/main.log")):
    log = f.read_text(encoding="utf-8", errors="replace")
    big = [round(float(m)) for m in re.findall(r"Overfull \\vbox \(([0-9.]+)pt too high\)", log) if float(m) > 20]
    if big: print(f.parent.name, big)
PY
```
넘친 프레임 찾기 — 한 프레임에 그림 2장 이상인 곳부터 본다.
```bash
python3 - <<'PY'
import re, pathlib
for f in sorted(pathlib.Path(".").glob("week*/main.tex")):
    s = f.read_text(encoding="utf-8")
    for m in re.finditer(r"\\begin\{frame\}(?:\[[^\]]*\])?(.*?)\\end\{frame\}", s, re.S):
        imgs = re.findall(r"includegraphics\[[^\]]*\]\{\./figs/([^}]+)\}", m.group(1))
        if len(imgs) >= 2:
            print(f"{f}:{s[:m.start()].count(chr(10))+1} {imgs}")
PY
```

관련: `image-sizing-overflow.md`(축 선택 규칙), `oversized-figure-and-column-budget.md`(columns 예산).
