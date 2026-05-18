# permissions/

이 디렉토리는 `~/.claude/settings.json` 의 `permissions` 섹션으로 병합될 권한 조각 파일들의 소스입니다.

## 구조

```
permissions/
├── README.md              설명 문서 (설치 대상 아님)
└── <name>.json            권한 조각 파일
```

## 조각 파일 포맷

```json
{
  "allow": ["WebFetch", "Bash(git log *)"],
  "deny": [],
  "ask": []
}
```

- `allow` / `deny` / `ask`: `settings.json` 의 해당 버킷에 추가될 권한 문자열 배열. 각 항목 포맷은 Claude Code의 permission 규칙(`Tool`, `Tool(arg)`, `Tool(pattern)` 등)을 따른다.
- 세 키 모두 선택 사항. 빈 배열이거나 생략 가능.
- 파일명 `<name>` 은 매니페스트에서 식별자로 사용됨. 저장소 내 중복 금지.
- 동일 권한 문자열은 저장소 내 **단 하나의 파일만** 선언한다.
- 권한 문자열에 `{{홈}}` 플레이스홀더를 사용할 수 있다. 설치 시 실제 홈 디렉토리 절대 경로로 치환된다.

## 병합 로직

설치 시 각 조각 파일의 항목마다:

1. `~/.claude/settings.json` 의 `permissions.<bucket>` 배열에 동일 문자열이 없으면 추가.
2. 이미 존재하면 중복 추가하지 않음.
3. 매니페스트(`~/.claude/.claude-setting-manifest.json`) 의 `permissions.<name>.entries` 에 등록.

## 참고

권한 규칙 문법은 Claude Code 공식 문서 permissions 섹션을 참조하십시오.
