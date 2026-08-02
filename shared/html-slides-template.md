# HTML 슬라이드 템플릿 (모드 E `/kgulec:web` 전용)

기존 `main.tex`(Beamer)를 **움직이는 HTML 프레젠테이션**(`slides.html`)으로 변환할 때 사용하는 표준 템플릿.
**아래 CSS·스켈레톤·빌드 스크립트를 그대로 사용한다. 간소화·임의 변경 금지** (`beamer-templates.md`와 동일 원칙).
실제 강의(경기대 특강, 56슬라이드)에서 브라우저 스크린샷으로 레이아웃 검증된 값이다.

- 엔진: reveal.js 5.x — 번들 위치 `${CLAUDE_PLUGIN_ROOT}/assets/web/` (`reveal.js`, `reveal.css`, `reset.css`). **CDN 요청 금지, 네트워크 불필요.**
- 출력: `slides.html` **단일 자립 파일** — reveal JS/CSS 인라인 + 이미지 전부 base64 data URI 임베드. 파일 하나만 복사하면 어디서든 발표.
- 해상도: 1280×720 (16:9).

## ⛔ 꽉 찬 레이아웃 규칙 (하드 룰 — 위반 금지)

Beamer PDF처럼 **슬라이드를 꽉 채운다. 위아래 죽은 여백 금지.** 이 규칙이 이 템플릿의 존재 이유 중 하나다.

1. **프레임 풀높이**: `.frame { height:676px }` (720 − 하단바 34 − 상단 패딩). **줄이지 않는다.**
2. **세로 배분**: `.fbody { justify-content:space-evenly }` + `.fbody > * { flex-shrink:0 }` — 콘텐츠가 적으면 간격이 벌어져 채우고, 많으면 gap(10px)이 최소 간격이 된다. `center` 정렬로 되돌리지 않는다.
3. **폰트 하한**: 아래 기준값(본문 22px, 박스 제목 22px, 표 19.5px, 코드 18.5px)에서 **내려서 여백을 만들지 않는다.** 콘텐츠가 적은 슬라이드는 폰트를 줄일 게 아니라 이미지·요소를 **키워서** 채운다.
4. **이미지는 크게**: 단독 이미지 슬라이드는 높이 330~400px 기준. 이미지가 작아 여백이 남으면 키운다.
5. **오버플로만 예외**: 콘텐츠가 676px을 넘칠 때만 그 슬라이드에 한해 인라인 `style="font-size:...px"`로 소폭(2~3px) 축소하거나 gap을 줄인다. 전역 축소 금지.
6. **검증 의무**: 변환 후 밀도 최저·최고 슬라이드를 각각 렌더 확인해 ①여백 과다 ②넘침 둘 다 없는지 본다(아래 「품질 검사」).

## 파일 구성 (빌드 방식)

한 파일로 직접 쓰면 수정이 어렵다. **파트 파일 + 빌드 스크립트** 방식을 쓴다:

```
작업폴더/
  main.tex               # 입력 (기존)
  web_src/               # 소스 (보존 — 이후 수정 시 재빌드)
    part1.html           # <head> + CSS + 슬라이드 전반부
    part2.html           # 슬라이드 중반부
    part3.html           # 슬라이드 후반부 + footbar + Reveal 초기화
    build.py             # 결합 + reveal 인라인 + 이미지 base64 → ../slides.html
    reveal.js reveal.css reset.css   # assets/web/에서 복사
  slides.html            # 출력 (단일 자립 파일)
```

reveal 번들 복사: macOS/Linux `cp "${CLAUDE_PLUGIN_ROOT}/assets/web/"* web_src/` · Windows `Copy-Item "$env:CLAUDE_PLUGIN_ROOT\assets\web\*" web_src\`.

## HTML 스켈레톤 (part1 머리 + part3 꼬리)

part1.html 시작:

```html
<!doctype html>
<html lang="ko">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>{과목명} — {주제}</title>
<style>
__RESET_CSS__
</style>
<style>
__REVEAL_CSS__
</style>
<style>
/* 아래 「표준 CSS」 전체를 여기에 */
</style>
</head>
<body>
<div class="reveal">
<div class="slides">
<!-- <section>들... -->
```

part3.html 끝 (슬라이드 뒤):

```html
</div><!-- /.slides -->

<!-- Madrid footline -->
<div id="footbar">
  <div class="f-auth">{과목명}</div>
  <div class="f-title">{주제}</div>
  <div class="f-num"><span id="pgnum">1</span> / <span id="pgtotal">N</span></div>
</div>

</div><!-- /.reveal -->

<script>
__REVEAL_JS__
</script>
<script>
Reveal.initialize({
  width: 1280, height: 720, margin: 0.0,
  minScale: 0.2, maxScale: 2.0,
  hash: true, history: true, center: false,
  progress: true, controls: true, controlsTutorial: false,
  slideNumber: false, transition: 'slide', backgroundTransition: 'fade',
  overview: true, touch: true,
  fragmentInURL: false, pdfSeparateFragments: false
});
function updateChrome() {
  var idx = Reveal.getIndices().h;
  document.getElementById('pgnum').textContent = idx + 1;
  document.getElementById('pgtotal').textContent = Reveal.getTotalSlides();
  document.body.classList.toggle('on-divider',
    Reveal.getCurrentSlide().classList.contains('divider'));
}
Reveal.on('ready', updateChrome);
Reveal.on('slidechanged', updateChrome);
</script>
</body>
</html>
```

## 표준 CSS (그대로 복사)

`:root` 4색은 **main.tex의 PRIMARY THEME 블록에서 읽어 치환**한다(`mqblue`→`--mqblue`, `mqdeepblue`→`--mqdeep`, `mqgrayblue`→`--mqgray`, `mqlightblue`→`--mqlight`). 의미색(example/warning/info/accent)은 고정.

```css
/* ================= THEME (Beamer Madrid 재현) ================= */
:root{
  --mqblue:#5B9BD5; --mqdeep:#2F5597; --mqgray:#4F81BD; --mqlight:#DDEBF7; /* ← main.tex PRIMARY THEME 매핑 */
  --exbg:#E2F0D9; --exfr:#38761D;
  --wnbg:#FFEBCC; --wnfr:#BF6400;
  --infbg:#EDEDED; --inffr:#646464;
  --ared:#C0392B; --apurple:#8E44AD; --ateal:#16A085;
  --ink:#2D2D2D; --cream:#FCF9F2;
}
html, body { font-family:'Pretendard','Apple SD Gothic Neo','Malgun Gothic','Noto Sans KR',sans-serif; }
.reveal { font-family:'Pretendard','Apple SD Gothic Neo','Malgun Gothic','Noto Sans KR',sans-serif; font-size:32px; color:var(--ink); }
.reveal .slides { text-align:left; }
.reveal .slides section { height:100%; padding:0; }
.reveal b, .reveal strong { font-weight:700; color:inherit; }
.reveal .progress { color:var(--mqblue); height:4px; }
.reveal .controls { color:var(--mqblue); }
.backgrounds { background:#fff; }

/* ---- 프레임 구조 (꽉 찬 레이아웃 하드 룰) ---- */
.frame { display:flex; flex-direction:column; height:676px; padding:20px 42px 0 42px; box-sizing:border-box; }
.ftitle { font-size:40px; font-weight:800; color:var(--mqdeep); margin:0 0 6px 0; padding-bottom:7px; border-bottom:3px solid var(--mqlight); letter-spacing:-0.01em; }
.fbody { flex:1; display:flex; flex-direction:column; justify-content:space-evenly; gap:10px; min-height:0; }
.fbody > * { flex-shrink:0; }
.lead { font-size:27px; font-weight:700; color:var(--ink); margin:2px 0; }
.center { text-align:center; }

/* ---- 하단 바 (Madrid footline) ---- */
#footbar { position:absolute; left:0; right:0; bottom:0; height:34px; display:flex; z-index:30; font-size:14.5px; transition:opacity .3s; }
#footbar .f-auth  { width:50%; background:var(--mqdeep);  color:#fff; display:flex; align-items:center; justify-content:center; font-weight:600; }
#footbar .f-title { width:40%; background:var(--mqlight); color:var(--mqgray); display:flex; align-items:center; justify-content:center; }
#footbar .f-num   { width:10%; background:var(--mqdeep);  color:var(--mqlight); display:flex; align-items:center; justify-content:center; }
body.on-divider #footbar { opacity:0; pointer-events:none; }

/* ---- tcolorbox 재현 ---- */
.box { border:1.6px solid; border-radius:7px; overflow:hidden; text-align:left; }
.box .btitle { font-weight:700; padding:6px 16px; font-size:22px; color:#fff; display:flex; align-items:center; gap:8px; }
.box .bbody { padding:11px 16px; font-size:22px; line-height:1.45; }
.box.concept { border-color:var(--mqdeep); } .box.concept .btitle { background:var(--mqdeep); } .box.concept .bbody { background:var(--mqlight); }
.box.example { border-color:var(--exfr); }  .box.example .btitle { background:var(--exfr); }  .box.example .bbody { background:var(--exbg); }
.box.warning { border-color:var(--wnfr); }  .box.warning .btitle { background:var(--wnfr); }  .box.warning .bbody { background:var(--wnbg); }
.box.info { border-color:var(--inffr); }    .box.info .btitle { background:var(--inffr); }    .box.info .bbody { background:var(--infbg); }
.box.mathbox { border-color:var(--mqdeep); background:#EAF2FA; padding:12px 18px; font-size:19px; }

/* ---- taskcard ---- */
.cards { display:flex; gap:16px; align-items:stretch; }
.cards .card { flex:1; background:#fff; border:2.2px solid; border-radius:8px; box-shadow:2px 3px 6px rgba(0,0,0,.14); padding:13px 10px; text-align:center; display:flex; flex-direction:column; align-items:center; justify-content:flex-start; gap:3px; }
.card .cicon { font-size:42px; line-height:1.15; }
.card .cname { font-size:21.5px; font-weight:800; }
.card .cdesc { font-size:17.5px; color:#555; line-height:1.4; }
.card.c-blue { border-color:var(--mqblue); }   .card.c-blue .cname { color:var(--mqblue); }
.card.c-purple { border-color:var(--apurple);} .card.c-purple .cname { color:var(--apurple); }
.card.c-green { border-color:var(--exfr); }    .card.c-green .cname { color:var(--exfr); }
.card.c-orange { border-color:var(--wnfr); }   .card.c-orange .cname { color:var(--wnfr); }
.card.c-teal { border-color:var(--ateal); }    .card.c-teal .cname { color:var(--ateal); }
.card.c-red { border-color:var(--ared); }      .card.c-red .cname { color:var(--ared); }
.card .bignum { font-size:50px; font-weight:900; line-height:1.1; }

/* ---- 2단 칼럼 ---- */
.cols { display:flex; gap:16px; align-items:stretch; }
.cols > div { min-width:0; }
.cols .col { flex:1; display:flex; flex-direction:column; gap:10px; justify-content:flex-start; }

/* ---- 리스트 ---- */
.reveal .bbody ul, .reveal .bbody ol { margin:0 0 0 4px; padding-left:24px; }
.reveal .bbody li { margin:4px 0; line-height:1.45; }

/* ---- 코드(verbatim) ---- */
.reveal pre.vb { margin:4px 0 0 0; padding:10px 14px; background:rgba(255,255,255,.72); border:1px solid rgba(0,0,0,.12); border-radius:5px; font-family:'SF Mono','D2Coding','Consolas','Menlo',monospace; font-size:18.5px; line-height:1.45; white-space:pre-wrap; word-break:break-all; box-shadow:none; width:auto; }

/* ---- 표 ---- */
.reveal table.tbl { border-collapse:collapse; margin:4px auto; font-size:19.5px; }
.reveal table.tbl th { background:var(--mqlight); color:var(--mqdeep); font-weight:800; padding:8px 14px; border-bottom:2.5px solid var(--mqdeep); text-align:center; }
.reveal table.tbl td { padding:7px 14px; border-bottom:1px solid #C9D6E8; vertical-align:top; line-height:1.35; }
.reveal table.tbl td:first-child { text-align:center; font-weight:700; border-right:1.5px solid #C9D6E8; }
.reveal table.tbl tr:last-child td { border-bottom:none; }

/* ---- 번호 원 (\numcircle 재현) ---- */
.nc { display:inline-flex; align-items:center; justify-content:center; width:25px; height:25px; border-radius:50%; background:var(--mqblue); color:#fff; font-weight:800; font-size:16px; vertical-align:-3px; margin-right:2px; }

/* ---- 섹션 디바이더 (\sectiondividerframe 재현) ---- */
.divwrap { position:relative; width:100%; height:720px; background:var(--mqdeep); }
.divwrap .band { position:absolute; top:110px; left:0; right:0; height:96px; background:var(--mqlight); }
.divwrap .panel { position:absolute; top:250px; left:75px; right:75px; bottom:80px; background:#fff; border-radius:4px; }
.divwrap .accent { position:absolute; top:250px; left:75px; width:26px; bottom:80px; background:var(--mqlight); border-radius:4px 0 0 4px; }
.divwrap .divc { position:absolute; top:250px; left:75px; right:75px; bottom:80px; display:flex; flex-direction:column; align-items:center; justify-content:center; gap:14px; }
.divc .divicon { font-size:88px; line-height:1; }
.divc .divtitle { font-size:52px; font-weight:900; color:var(--ink); letter-spacing:-0.01em; }
.divc .divsub { font-size:26px; color:var(--mqdeep); font-weight:600; }
.divc .divnum { position:absolute; top:-108px; font-size:24px; font-weight:800; color:var(--mqgray); letter-spacing:.22em; }
section.present .divicon { animation:divpop .7s cubic-bezier(.2,1.4,.4,1) both; }
section.present .divtitle { animation:divup .6s .18s ease-out both; }
section.present .divsub { animation:divup .6s .36s ease-out both; }
section.present .divnum { animation:divup .6s .5s ease-out both; }
@keyframes divpop { from { transform:scale(.2) rotate(-12deg); opacity:0; } to { transform:scale(1) rotate(0); opacity:1; } }
@keyframes divup { from { transform:translateY(26px); opacity:0; } to { transform:translateY(0); opacity:1; } }

/* ---- 타이틀 슬라이드 ---- */
.titlewrap { height:720px; display:flex; flex-direction:column; align-items:center; justify-content:center; gap:10px; text-align:center; background:linear-gradient(180deg,#fff 0%,var(--mqlight) 130%); }
.titlewrap img.logo { height:112px; margin-bottom:8px; }
.titlebox { background:var(--mqdeep); color:#fff; font-size:54px; font-weight:900; padding:16px 64px; border-radius:12px; box-shadow:0 6px 18px rgba(47,85,151,.35); }
.titlewrap .subtitle { font-size:30px; font-weight:700; color:var(--mqdeep); margin-top:16px; }
.titlewrap .presenter { font-size:23px; color:var(--ink); margin-top:4px; }
.titlewrap .dept { font-size:19px; color:#666; }
.titlewrap .date { font-size:18px; color:var(--mqgray); margin-top:10px; }
section.present .titlebox { animation:divup .7s .05s ease-out both; }
section.present .titlewrap .subtitle { animation:divup .6s .3s ease-out both; }
section.present .titlewrap .presenter, section.present .titlewrap .dept, section.present .titlewrap .date { animation:divup .6s .5s ease-out both; }
section.present .titlewrap img.logo { animation:divpop .8s ease-out both; }

/* ---- 목차 ---- */
.toc { display:flex; flex-direction:column; gap:12px; width:84%; margin:0 auto; }
.toc a { display:flex; align-items:center; gap:14px; background:#fff; border:1.8px solid var(--mqlight); border-radius:8px; padding:11px 20px; color:var(--ink); text-decoration:none; font-size:26px; font-weight:600; transition:transform .15s, border-color .15s; }
.toc a:hover { border-color:var(--mqblue); transform:translateX(6px); }
.toc .tnum { display:inline-flex; align-items:center; justify-content:center; min-width:34px; height:34px; border-radius:50%; background:var(--mqdeep); color:#fff; font-size:18px; font-weight:800; }
.toc .ticon { font-size:27px; }

/* ---- 단계 다이어그램 (간단 TikZ 화살표 흐름 재현) ---- */
.evo { display:flex; align-items:center; justify-content:center; gap:0; margin:10px 0; }
.evo .stage { border:2.4px solid; border-radius:8px; padding:16px 26px; text-align:center; font-size:20px; line-height:1.4; box-shadow:2px 3px 6px rgba(0,0,0,.15); }
.evo .stage b { display:block; font-size:24px; margin-bottom:2px; }
.evo .arrow { font-size:42px; font-weight:900; margin:0 12px; }
.evo .s1 { border-color:var(--mqblue); background:var(--mqlight); }
.evo .s2 { border-color:var(--apurple); background:#F3EAF8; }
.evo .s3 { border-color:var(--exfr); background:var(--exbg); }
.evo .a1 { color:var(--mqblue); } .evo .a2 { color:var(--exfr); }

/* ---- LIVE DEMO 배너 ---- */
.demobanner { border:2.4px solid var(--ared); border-radius:8px; background:#fff; box-shadow:2px 3px 7px rgba(0,0,0,.16); text-align:center; padding:12px; font-size:30px; font-weight:900; color:var(--ared); letter-spacing:.06em; }
section.present .demobanner { animation:demopulse 1.6s ease-in-out .4s 2; }
@keyframes demopulse { 0%,100% { transform:scale(1); } 50% { transform:scale(1.035); box-shadow:0 0 22px rgba(192,57,43,.45); } }

/* ---- 이미지 ---- */
.reveal .fbody img { max-width:100%; border:none; box-shadow:none; background:none; margin:0 auto; display:block; }
.imgshadow { border-radius:6px; box-shadow:0 4px 14px rgba(0,0,0,.18) !important; }

/* ---- 인쇄 / fragment ---- */
@media print { #footbar { position:fixed; } }
.reveal .fragment.fade-up { transition-duration:.45s; }
```

## Beamer → HTML 변환 매핑

| Beamer (main.tex) | HTML | 비고 |
|---|---|---|
| `\titlepage` | `<section data-transition="fade"><div class="titlewrap">…` | 로고 `kyonggi_logo.png` 임베드 |
| `\tableofcontents` | `.toc` — 섹션별 `<a href="#/N">` | N = 해당 디바이더의 0-기준 인덱스. **변환 후 인덱스 재검산** |
| `\sectiondividerframe{제목}{부제}{아이콘}` | `<section class="divider" data-transition="zoom"><div class="divwrap">…` | `divnum`에 SECTION 번호 |
| `\begin{frame}{제목}` | `<section><div class="frame"><h2 class="ftitle">제목</h2><div class="fbody">…` | |
| `tcolorbox` concept/example/warning/info | `<div class="box concept"><div class="btitle">…</div><div class="bbody">…</div></div>` | 색 매핑 그대로 |
| `taskcard=색` | `<div class="cards"><div class="card c-blue">…` | mqblue→c-blue, accentpurple→c-purple, examplefr→c-green, warningfr→c-orange, accentteal→c-teal, accentred→c-red |
| `columns` | `<div class="cols"><div class="col">…` | 비율은 `style="flex:0 0 40%"` |
| `tabular` | `<table class="tbl">` | 행 단위 fragment 가능 |
| `verbatim` | `<pre class="vb">` | **HTML 이스케이프 주의**(`<`, `>`, `&`) |
| `\numcircle{N}` | `<span class="nc">N</span>` | |
| `\faIcon{...}` | 이모지 1개 | robot→🤖, lightbulb→💡, exclamation-triangle→⚠️, check-circle→✅, play-circle→▶️ 등 의미 최근접 |
| TikZ 다이어그램 | HTML/CSS 재작성(단순 흐름은 `.evo` 패턴) 또는 인라인 SVG | 단계별 fragment 빌드업 부여 |
| `\textbf` / `\textit` | `<b>` / `<i>` | |
| `$\Rightarrow$` `$\sim$` `\ne` | ⇒ ~ ≠ | 수식 유니코드 치환. 복잡한 수식은 유니코드+`<sub>/<sup>` 조합 |
| LIVE DEMO taskcard | `<div class="demobanner">▶️ LIVE DEMO</div>` | |

## 애니메이션 규칙 (적당히 생동감 — 기본값)

- 슬라이드 전환 `slide`, 디바이더는 `data-transition="zoom"`, 타이틀은 `fade`.
- **첫 번째 콘텐츠 요소는 fragment 없이 바로 표시**, 이후 박스·카드는 `class="fragment fade-up"`으로 순차 등장.
- 표는 행(`<tr class="fragment fade-up">`), 핵심 교육 슬라이드는 `<li class="fragment fade-up">` 항목 단위.
- 디바이더·타이틀 등장 애니는 CSS `section.present` 셀렉터가 자동 처리(추가 작업 불필요).
- 순서 강제가 필요하면 `data-fragment-index="N"`.
- 과도 금지: 회전·바운스·타이핑 효과 등은 사용자가 "화려하게"를 요구할 때만.

## build.py (그대로 복사 후 경로만 수정)

```python
#!/usr/bin/env python3
"""slides.html 빌드: 파트 결합 + reveal 인라인 + 이미지 base64 임베드"""
import base64, os, re, sys

SRC = os.path.dirname(os.path.abspath(__file__))   # web_src/
OUT_DIR = os.path.dirname(SRC)                     # 작업 폴더 (main.tex 위치)
OUT = os.path.join(OUT_DIR, "slides.html")

def read(p):
    with open(p, encoding="utf-8") as f:
        return f.read()

html = "\n".join(read(os.path.join(SRC, f"part{i}.html")) for i in (1, 2, 3))
html = html.replace("__RESET_CSS__", read(os.path.join(SRC, "reset.css")))
html = html.replace("__REVEAL_CSS__", read(os.path.join(SRC, "reveal.css")))
html = html.replace("__REVEAL_JS__", read(os.path.join(SRC, "reveal.js")))

MIME = {".png": "image/png", ".jpg": "image/jpeg", ".jpeg": "image/jpeg", ".gif": "image/gif", ".svg": "image/svg+xml"}

def embed(m):
    rel = m.group(1)
    p = os.path.join(OUT_DIR, rel)
    if not os.path.exists(p):
        print(f"WARN: missing image {rel}", file=sys.stderr)
        return m.group(0)
    ext = os.path.splitext(p)[1].lower()
    data = base64.b64encode(open(p, "rb").read()).decode()
    return f'src="data:{MIME[ext]};base64,{data}"'

html, n = re.subn(r'src="((?:figs/)?[^":]+\.(?:png|jpe?g|gif|svg))"', embed, html)
print(f"embedded {n} images")

leftover = re.findall(r'__[A-Z_]+__', html)
if leftover:
    print(f"ERROR: unresolved placeholders {leftover}", file=sys.stderr)
    sys.exit(1)

with open(OUT, "w", encoding="utf-8") as f:
    f.write(html)
print(f"wrote {OUT} ({os.path.getsize(OUT)/1024/1024:.2f} MB)")
```

실행: macOS/Linux `python3 web_src/build.py` · Windows `py web_src\build.py` (또는 `python`).

## 품질 검사

1. **빌드 성공 + 이미지 임베드 수** 확인(스크립트 출력).
2. **렌더 검증** — `file://` 차단 도구가 있으므로 로컬 서버 사용: `python3 -m http.server 8471` 후 `http://localhost:8471/slides.html`.
   - Playwright(MCP 등) 가능하면: 브라우저에서 `Reveal.slide(h, 0, 99)`(fragment 전부 표시)로 이동하며 **①타이틀 ②디바이더 1개 ③밀도 최고 슬라이드 ④밀도 최저 슬라이드 ⑤마지막**을 스크린샷 확인 — 여백 과다·넘침·하단바 페이지번호 점검. 시각 확인은 Agent 위임 가능(메인 컨텍스트에 이미지 안 쌓이게).
   - **스크린샷 저장 위치는 `./.kgulec-tmp/`** — `shared/temp-files-policy.md` 준수(시스템 temp 금지·백신 오탐 방지). Playwright `filename`에 이 경로를 지정하고, Agent 위임 시에도 프롬프트에 명시한다.
   - 불가하면: 사용자에게 브라우저로 열어 위 5종을 확인해달라고 안내.
3. **TOC 링크 인덱스** 재검산(디바이더 위치 = 0-기준 슬라이드 번호).
4. 검사 끝나면 서버 종료(`kill $(lsof -ti :8471)` · Windows `Get-NetTCPConnection -LocalPort 8471 | % { Stop-Process -Id $_.OwningProcess }`) 후 **`./.kgulec-tmp/` 폴더째 삭제**(macOS/Linux `rm -rf ./.kgulec-tmp` · Windows `Remove-Item -Recurse -Force .kgulec-tmp`).

## 조작 안내 (사용자에게 마지막에 알려줄 것)

- 더블클릭으로 열림(네트워크 불필요). 방향키/스페이스 진행(클릭마다 요소 순차 등장), `ESC` 전체 오버뷰, `F` 전체화면.
- PDF 백업: `slides.html?print-pdf`로 연 뒤 브라우저 인쇄 → PDF 저장.
- 이후 수정: `web_src/` 파트 파일 수정 → `build.py` 재실행 (`/kgulec:web`이 담당).
