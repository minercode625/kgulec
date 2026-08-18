# FontAwesome5: 존재하지 않는 아이콘 이름

**증상** `! Package fontawesome5 Error: The requested icon <name> was not found.` → `Emergency stop`(즉시 중단).
**원인** 공식 목록에 없는 이름을 `\faIcon{...}`에 씀(예: `function`).
**해결** 실제 존재하는 이름만. 수학/함수 느낌 대체: `superscript`, `square-root-alt`, `code`. 새 아이콘은 사용 전 FontAwesome5 목록에서 이름 확인(직관적 이름이 아닐 수 있음).
**추가 사례** `bridge`(FA6 전용, FA5 없음) → 다리/랜드마크 느낌은 `archway` 사용. FA6에서 새로 생긴 이름은 FA5 패키지에 없는 경우가 많다. `folder-tree`(FA6 전용) → 계층/폴더구조 느낌은 `sitemap` 사용.

## 쓰기 전에 확인 (빌드 전 일괄 검사)

설치본 매핑 파일을 직접 grep 하면 추측이 필요 없다.

```bash
FA=$(kpsewhich fontawesome5-mapping.def)
for i in sitemap folder-tree user-shield; do
  grep -q "{$i}" "$FA" && echo "OK   $i" || echo "MISS $i"
done
```

새 아이콘을 여러 개 넣은 뒤에는 **컴파일 전에** 위 루프로 한 번에 거른다. 아이콘 하나가 `Emergency stop`이라 PDF가 아예 안 나온다.
