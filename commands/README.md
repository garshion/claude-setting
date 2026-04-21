# commands/

이 디렉토리는 `~/.claude/commands/` 로 설치될 slash command들의 소스입니다.

## 구조

각 command는 단일 markdown 파일입니다.

```
commands/
└── <command-name>.md
```

## 설치

저장소 루트 `CLAUDE.md` 의 "설치 절차 - Skills/Commands 설치 단계" 를 참조하십시오.
각 파일은 `~/.claude/commands/<command-name>.md` 로 복사되며, 매니페스트(`~/.claude/.claude-setting-manifest.json`)에 소스 해시가 기록됩니다.

## 참고

Command 파일 포맷과 frontmatter 필드 설명은 [../docs/extension-mechanisms.md](../docs/extension-mechanisms.md)의 "Slash Commands" 섹션을 참조하십시오.

최신 Claude Code는 skill 형식을 권장합니다. 단순 사용자 호출만 필요한 경우에도 `skills/<name>/SKILL.md` + `disable-model-invocation: true` 조합이 더 유연할 수 있습니다.
