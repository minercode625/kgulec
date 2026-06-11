# FontAwesome5: 존재하지 않는 아이콘 이름

**증상** `! Package fontawesome5 Error: The requested icon <name> was not found.` → `Emergency stop`(즉시 중단).
**원인** 공식 목록에 없는 이름을 `\faIcon{...}`에 씀(예: `function`).
**해결** 실제 존재하는 이름만. 수학/함수 느낌 대체: `superscript`, `square-root-alt`, `code`. 새 아이콘은 사용 전 FontAwesome5 목록에서 이름 확인(직관적 이름이 아닐 수 있음).
