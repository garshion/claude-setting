# skills/

이 디렉토리는 `~/.claude/skills/` 로 설치될 skill들의 소스입니다.

## 구조

각 skill은 자신의 이름으로 된 하위 디렉토리를 가지며, `SKILL.md` 파일이 필수입니다.

```
skills/
└── <skill-name>/
    ├── SKILL.md       필수. frontmatter + 지침 본문
    ├── reference.md   선택
    ├── examples/      선택
    └── scripts/       선택
```

## 설치

저장소 루트 `CLAUDE.md` 의 "설치 절차 - Skills/Commands 설치 단계" 를 참조하십시오.
각 skill 디렉토리는 `~/.claude/skills/<skill-name>/` 로 복사되며, 매니페스트(`~/.claude/.claude-setting-manifest.json`)에 소스 해시가 기록됩니다.

## 참고

Skill 파일 포맷과 frontmatter 필드 설명은 [../docs/extension-mechanisms.md](../docs/extension-mechanisms.md)의 "Skills" 섹션을 참조하십시오.
