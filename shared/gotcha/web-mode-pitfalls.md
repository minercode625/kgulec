# 모드 E(web) 함정 모음

## 본문 `______`(밑줄 블랭크)가 build.py placeholder 검사에 걸림
**증상** `ERROR: unresolved placeholders ['______']` → 빌드 실패(exit 1). 슬라이드 본문에 빈칸 표시용 밑줄 연속(`범인은 ______입니다`)을 썼을 때.
**원인** build.py의 잔여 placeholder 정규식 `__[A-Z_]+__`가 언더스코어 연속 문자열도 매치(`_`가 문자클래스에 포함).
**해결** 빈칸은 언더스코어 대신 밑줄 span으로: `<span style="display:inline-block;width:90px;border-bottom:3px solid var(--ink)"></span>`.

## cols에서 stretch된 .box 내부에 흰 여백
**증상** 2단 `.cols`에서 이미지 컬럼 옆의 단일 `.box`(flex:1)가 세로로 늘어나면, 텍스트 아래 박스 내부가 배경색 없이 흰색으로 남음. 에러 없음 — 렌더 검사로만 발견.
**원인** `.box`가 block이라 `.bbody` 높이가 텍스트만큼만 — 박스 잔여 공간은 투명(흰색).
**해결** 전역 CSS 보강: `.box{display:flex;flex-direction:column}` + `.box .bbody{flex:1}` → bbody 배경이 박스 끝까지 채움.

## `.box` flex 전환이 mathbox를 깨뜨림 (수식 글자 세로 나열)
**증상** mathbox 수식이 `P ( wt | w 1 …` 처럼 한 조각씩 세로로 쌓여 출력. 에러 없음.
**원인** 흰 여백 방지용 `.box{display:flex;flex-direction:column}`이 mathbox에도 적용 — mathbox는 btitle/bbody 없이 **인라인 수식을 직접 담는 박스**라서 인라인 조각들이 flex item으로 세로 배치됨.
**해결** 예외 규칙 추가: `.box.mathbox{display:block}`. (btitle/bbody 구조가 아닌 박스는 flex column 금지.)

## reset.css가 `<sub>/<sup>`를 평탄화 — 첨자가 본문 크기로
**증상** 수식 아래첨자(`w<sub>t</sub>`)가 본문과 같은 크기·기준선으로 렌더돼 `wt` 평문처럼 보임. computed style: `font-size` 부모와 동일, `vertical-align:baseline`.
**원인** reveal 번들 reset.css의 전역 리셋이 sub/sup 스타일 제거.
**해결** 복원 CSS 추가: `.reveal sub{font-size:.62em;vertical-align:-.22em}` / `.reveal sup{font-size:.62em;vertical-align:.45em}`.

## 재빌드 후 렌더 확인 시 브라우저 캐시
**증상** slides.html 재빌드 후 같은 URL 재방문하면 수정 전 화면이 보임(http.server는 캐시 헤더 없음).
**해결** 캐시버스터 쿼리로 확인: `slides.html?v=2#/24`.
