# 임시 파일 정책 (전 모드 공통 — 필수)

품질검사·미리보기 과정에서 생기는 임시 파일(PDF→PNG 렌더, 스크린샷, 샘플 컴파일 중간물 등)의 생성 위치와 정리 규칙.

## 왜

**시스템 임시 폴더(`/tmp`, `$TMPDIR`, `%TEMP%`) 사용 금지.** 백신(안티바이러스)이 설치된 환경에서는 시스템 temp에 파일을 만들고 읽고 실행하는 패턴이 **해킹(멀웨어) 행위로 오탐**되어 경고·격리·차단이 발생한다(특히 Windows). 교수 공용 플러그인이므로 어떤 보안 환경에서도 조용히 동작해야 한다.

## 규칙

1. **생성 위치**: 모든 임시 산출물은 **현재 작업 폴더 안의 `./.kgulec-tmp/`**에 만든다.
   - macOS/Linux: `mkdir -p ./.kgulec-tmp`
   - Windows: `New-Item -ItemType Directory -Force .kgulec-tmp`
2. **적용 대상**:
   - 오버플로·섹션 표지 검사용 PDF→PNG 렌더 (`pdftoppm -png -r 150 main.pdf ./.kgulec-tmp/out` 등)
   - 렌더 검증용 브라우저 스크린샷 (모드 E — Playwright 등 저장 경로를 이 폴더로 지정)
   - 셋업 샘플 테마의 컴파일 중간물 (`.tex`/`.aux`/`.log`/`.pdf` — 최종 `kgulec-theme-sample.png`만 작업 폴더로 복사)
   - 환경 점검에서 내려받는 설치 파일 (installer `.exe` 등 — `%TEMP%`에 받아 실행하면 백신 오탐 최다 패턴)
3. **Agent 위임 시에도 동일**: 시각 검사를 Agent에 위임할 때 **출력 경로로 `./.kgulec-tmp/`를 프롬프트에 명시**한다. "적당한 곳에 변환해라"라고 두면 Agent가 시스템 temp를 쓴다.
4. **정리(필수)**: 검사가 끝나고 문제가 없으면(수정 후 재검사 통과 포함) **`./.kgulec-tmp/` 폴더를 통째로 삭제**한다.
   - macOS/Linux: `rm -rf ./.kgulec-tmp`
   - Windows: `Remove-Item -Recurse -Force .kgulec-tmp`
   - 반복 검사 중에는 유지하고(재렌더 덮어쓰기), **작업 종료 시점에 반드시 삭제**. 검사 실패로 작업이 중단될 때도 남긴 이유를 사용자에게 알린다.
5. **예외 (삭제하지 않는 것)**:
   - 사용자 대면 산출물: `main.pdf`, `slides.html`, `kgulec-theme-sample.png`, `web_src/`(소스), `code/`
   - `main.tex` 컴파일의 정상 aux 파일(`main.aux`/`.log`/`.nav` 등) — 작업 폴더의 통상 산출물이며 이 정책 대상 아님
6. 작업 폴더가 git 저장소면 `.kgulec-tmp/`가 커밋되지 않게 주의한다(어차피 종료 시 삭제되지만, 중간 커밋 가능성이 있으면 `.gitignore`에 한 줄 추가를 제안).

## 예외: 작업 폴더가 클라우드 동기화 폴더일 때

작업 폴더가 iCloud/Dropbox/OneDrive 등 **동기화 폴더면 `./.kgulec-tmp/`도 동기화 대상**이라, 데몬이 쓰는 중인 파일을 가로채 손상시키는 문제(`gotcha/null-bytes-in-aux-cloud-synced-dir.md`)를 피할 수 없다. 이 경우 격리 빌드 위치로 **`~/.claude/kgulec/tmp/`**(Windows `%USERPROFILE%\.claude\kgulec\tmp\`)를 쓴다 — 동기화 밖이면서 시스템 temp도 아니라 두 문제(파일 손상·백신 오탐)를 모두 피한다. 사용 후 삭제 규칙은 동일.
