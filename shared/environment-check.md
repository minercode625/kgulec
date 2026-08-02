# 환경 점검 (TeX · PDF→PNG 도구)

슬라이드 컴파일과 테마/오버플로 미리보기에 필요한 외부 도구를 확인하고, 없으면 **사용자 동의 하에** 설치를 돕는다.

> **무음·자동 설치 금지.** TeX 배포판은 수백MB~수GB이고 시스템을 바꾸며 관리자 권한이 필요할 수 있다. 반드시 감지 → 명령 제시 → **동의** → 실행 순서로 진행한다.

## 필요 도구

- **`pdflatex`**(또는 `latexmk`) — `.tex` 컴파일. **필수**(없으면 진행 차단).
- **PDF→PNG** 중 하나: `pdftoppm`/`pdftocairo`(poppler) 또는 `magick`(ImageMagick) — 테마 미리보기·오버플로 검사용. **필수**(없으면 진행 차단).

두 도구(컴파일 + PDF→PNG) 모두 갖춰져야 `/kgulec:setup`이 다음 단계로 넘어간다.

## 1. 감지

- macOS/Linux(bash): `command -v pdflatex`, `command -v pdftoppm` 등
- Windows(PowerShell): `Get-Command pdflatex -ErrorAction SilentlyContinue`

컴파일 도구(`pdflatex`/`latexmk`)와 PDF→PNG 도구(`pdftoppm`/`pdftocairo`/`magick`)가 **둘 다** 있으면 → 통과(다음 단계로). 하나라도 없으면 → 2.

## 2. 없으면 — OS·패키지 매니저 확인 후 설치 제안 (동의 게이트)

실행 OS와 사용 가능한 패키지 매니저를 확인하고, 맞는 명령을 사용자에게 보여준 뒤 **동의를 받고** 실행한다. (관리자 권한·대용량 다운로드 가능함을 미리 알린다.)

| OS | 매니저 | 설치 명령 |
|----|--------|-----------|
| macOS | Homebrew | TeX: `brew install --cask mactex` · PNG: `brew install poppler` |
| macOS | Homebrew(경량) | `brew install --cask basictex` 후 `sudo tlmgr install kotex tcolorbox fontawesome5 algorithm2e pgf` |
| Windows | winget | PNG: `winget install poppler` (정상). MiKTeX: `winget install --id MiKTeX.MiKTeX` — **불완전 설치 주의**, 아래 「Windows MiKTeX 함정」 참고 |
| Windows | choco | `choco install miktex` · PNG: `choco install poppler` 또는 `imagemagick` |
| Linux(Debian/Ubuntu) | apt | `sudo apt install texlive-full poppler-utils` |
| Linux(Fedora) | dnf | `sudo dnf install texlive-scheme-full poppler-utils` |

설치 후 **재감지**하여 성공을 확인한다. 특히 Windows MiKTeX는 "설치 성공" 메시지가 떠도 실제로는 실패할 수 있으니, **반드시 `pdflatex --version`으로 검증**한다.

### Windows MiKTeX 함정 (검증된 설치 경로)

winget으로 MiKTeX를 깔면 "설치 성공"이라 떠도 **핵심 런타임 DLL(`MiKTeX###-core.dll` 등)이 누락된 채** 파일만 일부 풀려, `pdflatex` 실행 시 `0xC0000135 (DLL not found)`로 죽는 사례가 있다(`winget list`·레지스트리에도 미등록 = 실제 실패). 이때 검증된 복구 순서:

1. **불완전 설치 제거 + 공식 installer 내려받기** (PowerShell). installer는 `%TEMP%`가 아니라 **작업 폴더의 `./.kgulec-tmp/`**에 받는다 — `%TEMP%`에서 exe를 받고 실행하는 패턴은 백신이 멀웨어로 오탐하는 단골이다(`shared/temp-files-policy.md`):
   ```powershell
   Remove-Item "$env:LOCALAPPDATA\Programs\MiKTeX" -Recurse -Force -ErrorAction SilentlyContinue
   New-Item -ItemType Directory -Force .kgulec-tmp
   Invoke-WebRequest -Uri "https://miktex.org/download/win/basic-miktex-x64.exe" -OutFile ".kgulec-tmp\basic-miktex-x64.exe"
   ```
   (정확한 최신 installer URL은 https://miktex.org/download 에서 확인 — 버전 번호가 파일명에 들어간다.)
2. **무인 설치는 플래그를 `--unattended` 하나만** 쓴다:
   ```powershell
   Start-Process ".kgulec-tmp\basic-miktex-x64.exe" -ArgumentList "--unattended" -Wait
   ```
   - ⚠️ `--auto-install=yes` / `--shared=no` 등을 **함께 주면 비대화형 환경에서 거부되어 exit 1**(아무것도 안 깔림)이 난다. 단순화가 핵심.
3. **후처리** — 래퍼 링크 생성 + 누락 패키지 자동설치 활성화:
   ```powershell
   initexmf.exe --mklinks --force
   initexmf.exe --enable-installer
   initexmf.exe --set-config-value "[MPM]AutoInstall=1"
   ```
   (MiKTeX `bin`은 보통 사용자 PATH에 자동 등록된다. 안 되면 새 셸을 열거나 PATH를 갱신한다.)
4. **검증**: `pdflatex --version` → `MiKTeX-pdfTeX ...` 출력되면 성공. 성공 확인 후 내려받은 installer 정리: `Remove-Item -Recurse -Force .kgulec-tmp`.

> 요약: winget MiKTeX는 core DLL 누락으로 잘 실패한다 → 공식 installer를 `--unattended` **단독** 플래그로 돌리면 성공. poppler는 winget으로 한 번에 정상 설치된다.

## 3. 매니저가 없거나 동의하지 않으면 — 수동 안내 후 중단

공식 다운로드를 안내하고, **설치가 확인될 때까지 다음 단계로 진행하지 않는다**(아래 게이트 원칙 참고).

- macOS: MacTeX — https://tug.org/mactex/
- Windows: MiKTeX — https://miktex.org/ (또는 TeX Live)
- Linux/공통: TeX Live — https://tug.org/texlive/
- poppler — https://poppler.freedesktop.org · ImageMagick — https://imagemagick.org

설치를 마쳤으면 사용자가 다시 `/kgulec:setup`을 실행하도록 안내한다(재감지 후 통과하면 계속).

## 게이트 원칙 (`/kgulec:setup` — 차단)

`/kgulec:setup`에서는 컴파일 도구와 PDF→PNG 도구가 **둘 다 설치 확인**되어야 다음 단계(테마 선택·config 저장)로 넘어간다.

- 하나라도 없고 설치(동의 설치 또는 수동 설치)가 끝내 안 되면 → **setup을 여기서 중단**한다. 테마 선택·미리보기·config 저장을 진행하지 않는다.
- 재감지로 둘 다 확인되면 → 통과하여 다음 단계로.

> **모드 A/B/C(생성·수정·실습)는 다르다.** 그쪽은 비차단 — 도구가 없어도 `.tex`는 생성하고 "설치 후 `pdflatex main.tex`로 컴파일하세요"라고 안내한다. 컴파일·미리보기·오버플로 검사만 건너뛴다. 하드 게이트는 **setup 전용**이다.
