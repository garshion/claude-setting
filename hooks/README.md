# hooks/

이 디렉토리는 `~/.claude/settings.json` 의 `hooks` 섹션으로 병합될 hook 조각 파일들의 소스입니다.

## 구조

각 hook은 1파일 = 1 hook 블록(이벤트 + matcher 단위) 으로 표현됩니다.

```
hooks/
├── README.md              설명 문서 (설치 대상 아님)
└── <name>.json            hook 조각 파일
```

## 조각 파일 포맷

```json
{
  "event": "PreToolUse",
  "matcher": "WebSearch",
  "hooks": [
    {
      "type": "command",
      "command": "...",
      "timeout": 5
    }
  ]
}
```

- `event`: `PreToolUse`, `PostToolUse`, `SessionStart` 등 Claude Code hook 이벤트명.
- `matcher`: 해당 이벤트 배열에서 이 hook이 반응할 대상 패턴. `settings.json` 의 matcher 문법을 그대로 따름.
- `hooks`: 실행될 hook 실행자 목록. `settings.json` 의 해당 배열과 동일한 구조.
- 파일명 `<name>` 은 매니페스트에서 식별자로 사용됨. 저장소 내 중복 금지.
- 동일 `event` + `matcher` 조합에 대해 이 저장소가 제공하는 파일은 단 하나여야 함.

## 병합 로직

설치 시:

1. `~/.claude/settings.json` 의 `hooks[event]` 배열에서 동일 `matcher` 항목 검색.
2. 발견 시 해당 항목의 `hooks` 배열을 소스 내용으로 **교체**. 미발견 시 `{ matcher, hooks }` 블록을 배열에 **추가**.
3. 매니페스트(`~/.claude/.claude-setting-manifest.json`)의 `hooks.<name>` 항목으로 해시 추적.

## 참고

Hook 이벤트/매처/출력 스펙은 [../docs/extension-mechanisms.md](../docs/extension-mechanisms.md) 의 "Hooks" 섹션을 참조하십시오.
