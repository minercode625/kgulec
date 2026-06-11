# FontAwesome5: Invalid icon name `function`

## 증상
`\faIcon{function}` 사용 시 컴파일 에러 발생:
```
! Package fontawesome5 Error: The requested icon function was not found.
! Emergency stop.
```

## 원인
FontAwesome5 패키지에 `function`이라는 이름의 아이콘이 존재하지 않음. FontAwesome 공식 아이콘 목록에 없는 이름을 사용하면 컴파일이 즉시 중단됨.

## 해결 방법
유사한 대체 아이콘 사용:
- `\faIcon{superscript}` (수학/함수 느낌)
- `\faIcon{square-root-alt}` (수학 기호)
- `\faIcon{code}` (코드/함수)

## 예방
새로운 `\faIcon{...}` 사용 전 FontAwesome5 문서에서 아이콘 이름 확인. 특히 수학 관련 아이콘은 직관적 이름이 아닐 수 있음.
