# Claude 글로벌 설정 관리 저장소

이 저장소는 Claude Code의 글로벌 `CLAUDE.md` 설정을 관리합니다.
사용자가 환경 설정을 요청하면 아래 절차에 따라 `~/.claude/CLAUDE.md`를 생성합니다.

## 저장소 구조
- `common.md` - OS 무관 공통 설정 (코드 규칙, 커뮤니케이션 스타일 등)
- `platforms/` - OS별 설정 (windows.md, mac.md 등)
- `tools/` - 개발 도구별 설정 (vs2022.md, vs2026.md 등). `{{에디션}}` 플레이스홀더를 실제 값으로 치환하여 사용
- `skills/` - Claude Code skill 소스. 각 하위 디렉토리(`<skill-name>/SKILL.md` 등)가 `~/.claude/skills/<skill-name>/` 로 설치됨
- `commands/` - Claude Code slash command 소스. 각 `<command-name>.md` 파일이 `~/.claude/commands/<command-name>.md` 로 설치됨
- `hooks/` - Claude Code hook 조각 파일 소스. 각 `<name>.json` 파일이 `~/.claude/settings.json` 의 `hooks` 섹션으로 병합됨
- `permissions/` - Claude Code permissions 조각 파일 소스. 각 `<name>.json` 파일의 `allow/deny/ask` 항목이 `~/.claude/settings.json` 의 `permissions` 섹션으로 병합됨
- `docs/` - 설계·참고 문서

## 설치 절차

사용자가 환경 설정을 요청하면 다음 단계를 수행할 것:

### 1단계: OS 확인
- 현재 시스템의 OS를 확인한다.
- 해당하는 `platforms/*.md` 파일을 선택한다.

### 2단계: 개발 도구 탐지
- `tools/` 디렉토리의 각 도구 파일을 참고하여, 해당 도구가 설치되어 있는지 확인한다.
- 각 도구 파일의 "경로 탐지" 섹션에 따라:
  1. **기본 경로 확인**: 에디션별 기본 경로가 존재하는지 먼저 확인한다 (Enterprise → Professional → Community 순).
  2. **탐지 도구 사용**: 기본 경로에서 찾지 못한 경우, vswhere 등의 탐지 도구를 사용한다.
- 탐지된 도구의 에디션/경로 정보로 `{{에디션}}` 등의 플레이스홀더를 치환한다.
- 여러 버전이 발견되면 사용자에게 어떤 것을 사용할지 확인한다.

### 3단계: CLAUDE.md 모듈 설치
- 공통/플랫폼/도구별 지침을 연결(concat)하지 않고, `@` 임포트로 모듈화하여 설치한다.
- 설치 대상 구조:
  - `~/.claude/imports/claude-setting/common.md` ← 저장소 `common.md` 복사
  - `~/.claude/imports/claude-setting/platforms/<os>.md` ← 저장소 `platforms/<os>.md` 복사
  - `~/.claude/imports/claude-setting/tools/<tool>.md` ← 저장소 `tools/<tool>.md` 의 플레이스홀더 치환 완료본
  - `~/.claude/claude-setting.md` ← 저장소가 전담 관리하는 임포트 집약 파일
  - `~/.claude/CLAUDE.md` ← 사용자 파일. `@claude-setting.md` 한 줄만 보장, 그 외는 사용자 소유.
- 상세 규칙은 이 문서 하단의 "CLAUDE.md 모듈 설치 세부 규칙" 을 따른다.
- 기존 연결(concat) 방식으로 설치된 `~/.claude/CLAUDE.md` 가 감지되면 "전환 절차" 에 따라 사용자에게 모듈화 전환을 제안한다.

### 4단계: 저장소 경로 기록
- 전역 `~/.claude/CLAUDE.md`에 이 저장소의 경로를 기록한다.
- 형식: `# claude-setting 저장소 경로\n- <이 저장소의 절대 경로>`
- 이미 해당 섹션이 존재하면 경로가 현재 저장소와 일치하는지 확인하고, 다르면 현재 경로로 갱신한다.
- 이후 "전역 설정 변경 시 저장소 동기화" 절차에서 이 경로를 사용한다.

### 5단계: Skills/Commands 설치
- 저장소 `skills/` 디렉토리의 각 하위 디렉토리를 `~/.claude/skills/<skill-name>/` 에 설치한다.
- 저장소 `commands/` 디렉토리의 각 `<name>.md` 파일을 `~/.claude/commands/<name>.md` 에 설치한다.
- 설치 대상/매니페스트 처리/충돌 확인 등 상세 규칙은 이 문서 하단의 "Skills/Commands 설치 세부 규칙"을 따른다.
- `README.md` 등 설치 대상이 아닌 파일은 제외한다 (상세 규칙 참조).

### 6단계: Hooks 설치
- 저장소 `hooks/` 디렉토리의 각 `<name>.json` 파일을 `~/.claude/settings.json` 의 `hooks` 섹션에 병합한다.
- 상세 규칙은 이 문서 하단의 "Hooks 설치 세부 규칙"을 따른다.
- `README.md` 등 설치 대상이 아닌 파일은 제외한다 (상세 규칙 참조).

### 7단계: Permissions 설치
- 저장소 `permissions/` 디렉토리의 각 `<name>.json` 파일의 항목들을 `~/.claude/settings.json` 의 `permissions.allow` / `permissions.deny` / `permissions.ask` 배열에 병합한다.
- 상세 규칙은 이 문서 하단의 "Permissions 설치 세부 규칙"을 따른다.
- `README.md` 등 설치 대상이 아닌 파일은 제외한다 (상세 규칙 참조).

### 8단계: 결과 보고
- 적용된 설정 요약을 사용자에게 보고한다 (OS, 탐지된 도구, 에디션, 경로, 설치/갱신된 skill·command·hook·permission 개수 등).

## 업데이트 절차

사용자가 설정 최신화/업데이트를 요청하면 (예: "업데이트해줘", "설정 최신화해줘", "최신 설정으로 맞춰줘" 등) 다음 간소화된 절차를 수행할 것:

- 이 절차는 이미 설치 절차를 통해 `~/.claude/CLAUDE.md`가 구성된 환경에서 사용한다.
- OS 탐지, 도구 탐지를 다시 수행하지 않고, 기존 설정에서 환경 정보를 파악한다.

### 1단계: 기존 설정 읽기
- `~/.claude/CLAUDE.md`를 읽어 현재 적용된 OS, 도구 정보를 파악한다.
- 파일이 없으면 업데이트 대상이 없으므로, 설치 절차를 먼저 실행하라고 안내한다.

### 2단계: CLAUDE.md 모듈 갱신
- 설치 절차 3단계와 동일하게 `~/.claude/imports/claude-setting/` 의 개별 임포트 파일과 `~/.claude/claude-setting.md` 를 갱신한다.
- 상세 규칙은 "CLAUDE.md 모듈 설치 세부 규칙" 을 따른다.
- 기존 연결(concat) 방식이 감지되면 "전환 절차" 를 사용자에게 제안한다.

### 3단계: Skills/Commands 갱신
- 설치 절차의 5단계와 동일하게 `skills/`, `commands/` 소스를 `~/.claude/skills/`, `~/.claude/commands/` 에 반영한다.
- 상세 규칙은 "Skills/Commands 설치 세부 규칙" 을 따른다.

### 4단계: Hooks 갱신
- 설치 절차의 6단계와 동일하게 `hooks/` 소스를 `~/.claude/settings.json` 의 `hooks` 섹션에 반영한다.
- 상세 규칙은 "Hooks 설치 세부 규칙" 을 따른다.

### 5단계: Permissions 갱신
- 설치 절차의 7단계와 동일하게 `permissions/` 소스를 `~/.claude/settings.json` 의 `permissions` 섹션에 반영한다.
- 상세 규칙은 "Permissions 설치 세부 규칙" 을 따른다.

### 6단계: 결과 보고
- 변경된 내용을 요약하여 사용자에게 보고한다 (갱신된 skill·command·hook·permission 포함). 변경사항이 없으면 "이미 최신 상태"임을 안내한다.

## Skills/Commands 설치 세부 규칙

설치 절차 6단계 및 업데이트 절차 4단계의 공통 규칙이다.

### 설치 대상

| 저장소 소스 | 설치 대상 |
|---|---|
| `skills/<name>/` 디렉토리 (각 `<name>`별 하위 디렉토리) | `~/.claude/skills/<name>/` 로 **디렉토리 전체** 복사 |
| `commands/<name>.md` 파일 | `~/.claude/commands/<name>.md` 로 **파일** 복사 |

- `skills/README.md`, `commands/README.md` 처럼 각 디렉토리 **루트에 있는 `README.md`** 는 설치 대상에서 제외한다 (저장소 설명 문서).
- `skills/<name>/` **하위의** 파일은 이름과 무관하게 전부 설치 대상이다 (`SKILL.md` 외 `reference.md`, `examples/`, `scripts/` 등 포함).

### 매니페스트

**위치**: `~/.claude/.claude-setting-manifest.json` (장비별 로컬 상태. 저장소에 포함되지 않음)

**포맷 (버전 1)**:
```json
{
  "version": 1,
  "skills": {
    "<skill-name>": {
      "files": {
        "<상대 경로>": "<sha256>"
      },
      "installed_at": "<ISO 8601 UTC>"
    }
  },
  "commands": {
    "<command-name>": {
      "hash": "<sha256>",
      "installed_at": "<ISO 8601 UTC>"
    }
  }
}
```

- `skills.<name>.files` 의 키는 skill 디렉토리 내부 상대 경로(예: `SKILL.md`, `scripts/validate.sh`). 값은 **설치 시점의 소스 파일 SHA-256**.
- 해시는 소스 파일 바이트 기반 SHA-256. 줄바꿈 정규화 등 변형을 하지 않는다.
- 파일이 없으면 새로 생성한다. 읽을 때 JSON 파싱 실패 시 사용자에게 보고하고 중단한다 (임의 재작성 금지).
- 향후 Hooks/Permissions 설치도 같은 파일에 `hooks`, `permissions` 키로 확장될 예정이다.

### 설치/갱신 의사결정 (각 파일 단위)

매니페스트 단위는 skill/command이지만, 충돌 판정은 개별 파일 해시로 수행한다.

skill의 각 내부 파일 또는 command 단일 파일에 대해:

1. **대상 파일이 없음**
   → 복사 후 매니페스트에 현재 소스 해시 기록.

2. **대상 파일이 있고, 매니페스트에도 해당 경로 기록이 있음**
   - 현재 대상 파일 해시 == 매니페스트 기록 해시 → **저장소 설치본 그대로**. 소스와 내용이 다르면 조용히 덮어쓰고 매니페스트 해시를 새 소스 해시로 갱신.
   - 현재 대상 파일 해시 != 매니페스트 기록 해시 → **사용자가 수정한 것**. 사용자에게 덮어쓸지 건너뛸지 물어본다.

3. **대상 파일이 있지만 매니페스트에 기록 없음**
   → **외부 출처(다른 저장소/수동 작성)**. 사용자에게 덮어쓸지 건너뛸지 물어본다. 덮어쓰기 선택 시 매니페스트에 새로 기록한다.

4. **매니페스트에 기록은 있지만 대상 파일이 실제로는 없음**
   → 수동 삭제된 것으로 간주하여 매니페스트 항목을 제거하고 1번 규칙으로 새로 설치한다.

### 충돌 시 사용자 확인 방식

- 옵션 목록을 나열하지 말고, 대화형 문장으로 상황을 설명한 뒤 "덮어쓸지 건너뛸지" 를 물어본다.
- 필요 시 소스와 현재 대상 파일의 diff를 제시할 수 있음을 안내한다.
- 같은 세션 내 "모두 덮어쓰기" 같은 일괄 응답을 사용자가 먼저 지시하지 않는 한, 파일마다 개별 확인한다.

### 저장소 소스에서 제거된 항목 처리

- 저장소 `skills/` 또는 `commands/` 에서 삭제된 항목이 매니페스트에는 남아 있는 경우, **자동으로 제거하지 않는다**.
  - 사용자에게 "소스에서 사라진 항목이 있다" 고 보고만 하고, 제거 여부는 사용자 요청 시에만 수행.
  - 기본적으로 언인스톨 기능은 제공하지 않는다 (다장비 동기화 편의성 우선).

### 실행 환경 고려사항

- 경로 구분자는 각 OS에 맞게 처리한다. 매니페스트에 기록하는 skill 내부 상대 경로는 **POSIX 스타일 슬래시 `/`** 로 통일한다.
- SHA-256은 Node/Python/PowerShell/shasum 등 어느 도구든 사용 가능. 실행 도구 선택은 환경에 맡긴다.
- 설치/갱신된 파일은 줄바꿈 변환을 수행하지 않고 소스 바이트 그대로 복사한다.

## Hooks 설치 세부 규칙

설치 절차 7단계 및 업데이트 절차 5단계의 공통 규칙이다.

### 설치 대상

| 저장소 소스 | 설치 대상 |
|---|---|
| `hooks/<name>.json` 파일 | `~/.claude/settings.json` 의 `hooks[<event>]` 배열 안의 `{matcher, hooks}` 블록 |

- `hooks/README.md` 는 설치 대상에서 제외한다.
- 파일명 `<name>` 은 매니페스트 식별자로만 사용된다. 병합 위치는 파일 내용의 `event` + `matcher` 로 결정된다.

### 조각 파일 포맷

```json
{
  "event": "PreToolUse",
  "matcher": "WebSearch",
  "hooks": [
    { "type": "command", "command": "...", "timeout": 5 }
  ]
}
```

- `event`: `PreToolUse`, `PostToolUse`, `SessionStart`, `Stop` 등 Claude Code hook 이벤트명.
- `matcher`: 해당 이벤트 배열 안에서 반응할 대상 패턴(`settings.json` 의 matcher 문법 준수).
- `hooks`: `settings.json` 내 동일 위치에 들어갈 실행자 배열.
- 동일 `event` + `matcher` 조합에 대해 저장소가 제공하는 파일은 단 하나여야 한다.

### 병합 동작

각 조각 파일에 대해:

1. `~/.claude/settings.json` 이 없으면 빈 객체로 생성한다.
2. `settings.json.hooks[event]` 배열을 보장한다(없으면 빈 배열).
3. 배열에서 `matcher` 가 동일한 블록을 찾는다.
   - 발견 시: 해당 블록의 `hooks` 배열을 소스 값으로 **교체**(matcher는 유지).
   - 미발견 시: `{ "matcher": <matcher>, "hooks": <hooks> }` 블록을 배열 끝에 **추가**.
4. 기존 사용자가 작성한 다른 hook 블록이나 다른 event 설정은 건드리지 않는다.

### 매니페스트

`~/.claude/.claude-setting-manifest.json` 의 `hooks` 섹션에 기록한다:

```json
"hooks": {
  "<name>": {
    "event": "PreToolUse",
    "matcher": "WebSearch",
    "hash": "<sha256>",
    "installed_at": "<ISO 8601 UTC>"
  }
}
```

- `hash`: 병합 후 `settings.json` 안에 들어간 `{matcher, hooks}` 블록을 **canonical JSON** 으로 직렬화한 뒤 SHA-256 을 계산한 값.
  - canonical JSON 규칙: 객체 키 알파벳 정렬, 공백 제거(`JSON.stringify` 기준 separator 없음), UTF-8.
  - 배열 원소 순서는 소스 순서를 유지한다(알파벳 정렬 대상 아님).

### 설치/갱신 의사결정

각 조각 파일(`<name>`)에 대해, 현재 `settings.json` 안에서 `event` + `matcher` 매칭 블록을 찾은 뒤 canonical JSON → SHA-256 을 계산해 매니페스트 `hooks.<name>.hash` 와 비교한다.

1. **매칭 블록 없음, 매니페스트 기록도 없음**
   → 새로 병합, 매니페스트에 새 해시 기록.

2. **매칭 블록 있음, 매니페스트 기록 있음**
   - 현재 블록 해시 == 매니페스트 해시 → **저장소 설치본 그대로**. 소스 내용이 다르면 조용히 교체 후 매니페스트 해시 갱신.
   - 현재 블록 해시 != 매니페스트 해시 → **사용자가 수정한 것**. 덮어쓸지 건너뛸지 물어본다.

3. **매칭 블록 있음, 매니페스트 기록 없음**
   → **외부 출처(수동 작성/다른 저장소)**. 덮어쓸지 건너뛸지 물어본다. 덮어쓰기 선택 시 매니페스트에 새로 기록한다.

4. **매칭 블록 없음, 매니페스트 기록 있음**
   → 사용자가 수동으로 삭제한 것으로 간주. 매니페스트 항목을 제거하고 1번 규칙으로 새로 병합한다.

### 충돌 시 사용자 확인 방식

- Skills/Commands 와 동일한 대화형 방식. 옵션 목록을 나열하지 말고 상황을 설명한 뒤 "덮어쓸지 건너뛸지" 를 물어본다.
- 필요 시 현재 블록과 소스의 diff를 제시할 수 있음을 안내한다.

### 저장소 소스에서 제거된 항목 처리

- 저장소 `hooks/` 에서 삭제된 항목이 매니페스트에는 남아 있는 경우, **자동 제거하지 않는다**.
  - 사용자에게 "소스에서 사라진 hook이 있다"고 보고만 하고, 제거는 사용자 요청 시에만 수행.

### 실행 환경 고려사항

- `settings.json` 을 쓸 때는 기존 들여쓰기 스타일(보통 2칸)을 유지한다. 키 순서는 가능하면 기존 구조를 보존한다.
- canonical JSON 계산 시 Node `JSON.stringify` 의 기본 출력(이스케이프 포함)을 사용한다. 플랫폼 간 동일 결과가 나오도록 UTF-8 바이트 기반으로 SHA-256 을 구한다.
- WebSearch hook 이전 전환 시점: 기존 `settings.json` 에 매니페스트 기록 없는 WebSearch hook 이 있을 수 있다. 이 경우 3번 규칙에 따라 사용자에게 확인 후 매니페스트에 등록한다.

## Permissions 설치 세부 규칙

설치 절차 7단계 및 업데이트 절차 5단계의 공통 규칙이다.

### 설치 대상

| 저장소 소스 | 설치 대상 |
|---|---|
| `permissions/<name>.json` 파일 | `~/.claude/settings.json` 의 `permissions.allow` / `permissions.deny` / `permissions.ask` 배열 |

- `permissions/README.md` 는 설치 대상에서 제외한다.
- 파일명 `<name>` 은 매니페스트 식별자.
- 동일 권한 문자열을 저장소 내 여러 파일이 선언하지 않도록 한다(중복 금지).

### 조각 파일 포맷

```json
{
  "allow": ["WebFetch", "Bash(git log *)"],
  "deny": [],
  "ask": []
}
```

- `allow` / `deny` / `ask` 셋 모두 선택 사항. 생략하거나 빈 배열로 둘 수 있다.
- 각 문자열은 Claude Code permission 규칙 문법(예: `Tool`, `Tool(arg)`, `Tool(pattern)`)을 따른다.

### 병합 동작

각 조각 파일의 각 항목 `E` (버킷 `B`)에 대해:

1. `~/.claude/settings.json` 이 없으면 빈 객체로 생성하고 `permissions.<B>` 배열을 보장한다.
2. 해당 배열에 `E` 가 이미 있으면 중복 추가하지 않는다.
3. 없으면 배열 끝에 `E` 를 추가한다.
4. 기존 사용자가 추가한 다른 항목은 건드리지 않는다. 순서도 변경하지 않는다.

### 매니페스트

`~/.claude/.claude-setting-manifest.json` 의 `permissions` 섹션에 기록한다:

```json
"permissions": {
  "<name>": {
    "entries": {
      "allow": ["WebFetch"],
      "deny": [],
      "ask": []
    },
    "installed_at": "<ISO 8601 UTC>"
  }
}
```

- `entries` 는 해당 조각 파일이 소유하는 항목 목록. 소스 파일의 선언 내용을 그대로 기록(빈 배열은 생략 가능).
- Skills/Hooks와 달리 SHA-256 해시는 기록하지 않는다. 권한 항목은 단순 문자열이므로 값 자체를 매니페스트에 보관해 비교한다.

### 설치/갱신 의사결정

각 조각 파일 `<name>` 의 각 항목 `E` (버킷 `B`)에 대해 독립적으로 판단한다.

1. **`E` 가 settings에 없음, 매니페스트에도 없음**
   → 배열에 추가, 매니페스트에 등록.

2. **`E` 가 settings에 있음, 매니페스트에 있음**
   → 이미 설치됨. 변경 없음.

3. **`E` 가 settings에 있음, 매니페스트에 없음**
   → 사용자/외부가 먼저 추가한 동일 값. 매니페스트에 조용히 등록(충돌 아님, 값이 같으므로 사용자에게 묻지 않는다). 이후부터는 저장소 소유로 추적.

4. **`E` 가 settings에 없음, 매니페스트에 있음**
   → 사용자가 수동으로 제거한 것으로 간주. 매니페스트 항목을 제거하고 1번 규칙으로 재추가한다.
   - 권한 제거는 보안상 의도를 존중하는 편이 안전하지만, 다장비 동기화가 주 목적이므로 저장소 선언을 우선한다. 필요 시 해당 소스 파일 자체를 저장소에서 제거해야 한다.

### 저장소 소스에서 제거된 항목 처리

- 저장소 `permissions/` 에서 삭제되었거나, 특정 조각 파일의 `entries` 에서 빠진 항목이 매니페스트에는 남아 있는 경우, **자동 제거하지 않는다**.
  - 사용자에게 "소스에서 사라진 permission이 있다" 고 보고만 하고, 제거는 사용자 요청 시에만 수행.

### 실행 환경 고려사항

- `settings.json` 을 쓸 때는 기존 들여쓰기 스타일(보통 2칸)을 유지한다.
- WebFetch permission 이전 전환 시점: 기존 `settings.json` 에 매니페스트 기록 없는 `WebFetch` 항목이 있을 수 있다. 3번 규칙에 따라 조용히 매니페스트에 등록되므로 사용자 추가 확인 없이 전환된다.

## CLAUDE.md 모듈 설치 세부 규칙

설치 절차 3단계 및 업데이트 절차 2단계의 공통 규칙이다.

### 설치 구조 (두 단계 임포트)

```
~/.claude/
├── CLAUDE.md                             사용자 파일. 저장소 개입 최소화.
├── claude-setting.md                     저장소 전담. 임포트 집약 파일.
└── imports/
    └── claude-setting/
        ├── common.md                     common.md 원본 복사
        ├── platforms/
        │   └── <os>.md                   선택된 플랫폼 파일 원본 복사
        └── tools/
            └── <tool>.md                 플레이스홀더 치환 완료본
```

- Claude Code 의 `@` 임포트는 최대 5단계 재귀 허용 — 두 단계 임포트에 문제없음.
- 저장소가 관리하는 파일은 `claude-setting.md` 와 `imports/claude-setting/**` 전체. `~/.claude/CLAUDE.md` 자체는 "한 줄 + 경로 섹션" 외에는 저장소가 건드리지 않는다.

### ~/.claude/CLAUDE.md 에 대한 개입 범위

저장소가 보장해야 하는 항목은 두 가지뿐이다:

1. **`@claude-setting.md` 임포트 라인**이 파일 상단(맨 위 ~ 초반 섹션 이전)에 존재.
   - 없으면 파일 최상단에 삽입.
   - 있으면 그대로 유지(위치·앞뒤 공백 포함).
2. **`# claude-setting 저장소 경로` 섹션**이 존재하고 현재 저장소 절대 경로와 일치.
   - 설치 절차 4단계(저장소 경로 기록)의 규칙을 그대로 사용.

그 외의 어떤 줄·섹션도 저장소가 임의로 추가·제거·수정하지 않는다.

### claude-setting.md 의 내용

저장소가 전체를 재작성한다. 형식:

```markdown
# claude-setting 임포트 집약 파일

이 파일은 `claude-setting` 저장소에 의해 자동 생성됩니다. 직접 편집하지 마십시오.
사용자 고유 지침은 `~/.claude/CLAUDE.md` 에 작성하십시오.

@imports/claude-setting/common.md
@imports/claude-setting/platforms/<os>.md
@imports/claude-setting/tools/<tool>.md
```

- 임포트 순서는 `common` → `platforms/<os>` → `tools/<tool>` (기존 연결 순서와 동일).
- 도구/플랫폼이 여러 개 선택된 경우 각각 한 줄씩.

### 플레이스홀더 치환

- `common.md`, `platforms/*.md` 는 원본을 그대로 복사.
- `tools/*.md` 는 설치 시점에 `{{에디션}}` 등 플레이스홀더를 탐지된 값으로 치환한 후 복사. 치환된 바이트가 설치 대상.

### 매니페스트

기존 매니페스트에 `claudeMdAggregator` 와 `imports` 키를 추가한다:

```json
{
  "claudeMdAggregator": {
    "path": "claude-setting.md",
    "hash": "<sha256>",
    "installed_at": "<ISO 8601 UTC>"
  },
  "imports": {
    "common.md": {
      "target": "imports/claude-setting/common.md",
      "hash": "<sha256>",
      "installed_at": "<ISO 8601 UTC>"
    },
    "platforms/windows.md": {
      "target": "imports/claude-setting/platforms/windows.md",
      "hash": "<sha256>",
      "installed_at": "<ISO 8601 UTC>"
    },
    "tools/vs2026.md": {
      "target": "imports/claude-setting/tools/vs2026.md",
      "hash": "<sha256-치환-완료본-기준>",
      "installed_at": "<ISO 8601 UTC>"
    }
  }
}
```

- `imports.<key>.key` 는 저장소 기준 상대 경로(POSIX 슬래시).
- `imports.<key>.target` 은 `~/.claude` 기준 상대 경로(POSIX 슬래시).
- `imports.<key>.hash` 는 실제로 **설치된 대상 파일(치환 완료본)** 의 SHA-256.

### 설치/갱신 의사결정

**`claude-setting.md` 에 대해**:
- Skills/Commands 와 동일한 4단 규칙(파일 존재·매니페스트 존재·해시 일치 여부). 사용자가 편집하지 않는 것이 원칙이지만, 편집된 경우 덮어쓸지 건너뛸지 물어본다.

**`imports/claude-setting/<path>` 각 파일에 대해**:
- Skills/Commands 와 동일한 4단 규칙. 해시 비교로 사용자 수정 감지.
- 도구 파일의 경우 플레이스홀더 치환 결과가 환경에 따라 달라질 수 있음 — 매니페스트 해시 기준으로 판단하면 같은 장비에서 재설치 시 문제없음.

**`~/.claude/CLAUDE.md` 에 대해**:
- 파일이 없으면 새로 생성(`@claude-setting.md` + 경로 섹션만 포함).
- 있으면 "개입 범위" 2개만 검사·갱신. 해시·매니페스트 추적 대상 아님.

### 전환 절차 (기존 연결 방식에서 모듈형으로)

기존 `~/.claude/CLAUDE.md` 가 연결(concat) 방식으로 설치된 경우, 매니페스트에 `imports` 키가 없고 파일에 `@claude-setting.md` 라인이 없는 상태다. 이 경우:

1. **전환 여부를 사용자에게 확인**. 설치 절차를 진행하지 않고 물어봄.
   - 거부 시: 모듈화 단계를 건너뛰고 기존 파일을 그대로 둔다. Skills/Commands, Hooks, Permissions 단계는 정상 진행.
2. **승인 시**:
   1. `~/.claude/CLAUDE.md` 를 `~/.claude/CLAUDE.md.backup-<ISO8601 UTC 타임스탬프>` 로 복사(백업).
   2. 사용할 소스 파일들(`common.md`, 선택된 `platforms/<os>.md`, 선택된 `tools/<tool>.md`)의 **최상위 `# ` 헤딩 텍스트 목록**을 수집한다.
   3. `~/.claude/CLAUDE.md` 를 파싱하여 각 최상위 `# ` 섹션을 식별한다. 헤딩 텍스트가 2번 목록에 있으면 해당 섹션 전체(다음 `# ` 직전까지)를 제거 대상으로 표시.
   4. 제거 대상을 뺀 나머지 내용을 유지한 채, 파일 최상단에 `@claude-setting.md` 라인을 삽입.
   5. `# claude-setting 저장소 경로` 섹션은 유지(없으면 4단계에서 추가).
   6. 사용자에게 **"소스 파일과 동일한 헤딩을 가진 섹션은 제거되었으며, 해당 섹션에 사용자가 직접 수정해 둔 내용이 있었다면 백업 파일에서 확인 필요"** 라고 명시적으로 고지.
3. 전환 이후는 일반 모듈 설치 규칙을 따른다.

### 충돌 시 사용자 확인 방식

- Skills/Commands 와 동일한 대화형 방식.
- 특히 `claude-setting.md` 수정 감지 시: "이 파일은 저장소가 자동 관리하는 집약 파일입니다. 수정 사항을 덮어써도 될까요?" 라는 식으로 상황을 명시.

### 저장소 소스에서 제거된 항목 처리

- 저장소에서 삭제된 소스 파일이 매니페스트의 `imports` 에 남아 있는 경우:
  - 자동 삭제하지 않는다.
  - 사용자에게 보고 후 요청 시에만 `~/.claude/imports/claude-setting/<path>` 와 매니페스트 항목을 제거.
  - `claude-setting.md` 의 `@` 임포트 라인도 그에 맞춰 재생성되어 자동으로 갱신된다(이 파일은 전체 재작성이 원칙이므로).

### 실행 환경 고려사항

- 경로 구분자는 OS별로 적절히 처리. 매니페스트에는 POSIX 슬래시로 통일 기록.
- 파일 복사 시 줄바꿈 변환을 수행하지 않고 소스 바이트 그대로 복사(플레이스홀더 치환 뒤 바이트).
- `@` 임포트 라인은 **POSIX 슬래시**(`@imports/claude-setting/...`)로 작성. Windows 환경에서도 Claude Code 는 이를 해석함.
