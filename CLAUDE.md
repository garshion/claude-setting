# Claude 글로벌 설정 관리 저장소

이 저장소는 Claude Code의 글로벌 `CLAUDE.md` 설정과 skill/command/hook/permission 을 관리한다.
설치/업데이트는 **이 저장소 디렉토리에서 직접 요청**한다(저장소 경로를 따로 기록하지 않는다).

설치/업데이트의 기본 동작은 **단순 덮어쓰기**다. 매니페스트·해시·충돌 판정을 사용하지 않는다.
원칙: 설치본(`~/.claude`)을 직접 편집하지 않는다. 고칠 내용은 저장소 소스를 고치고 재설치한다.

## 저장소 구조
- `common.md` - 항상 로드되는 최소 기본 지침
- `coding-guidelines.md` - 코딩 가이드라인 (항상 로드)
- `conventions/` - 언어별 코드 규칙 (cpp.md, csharp.md 등. 온디맨드)
- `platforms/` - OS별 환경 탐지 대상 (windows.md, mac.md, linux.md. 온디맨드)
- `tools/` - 개발 도구별 설정 (vs2022.md, vs2026.md 등. `{{에디션}}` 치환. 온디맨드)
- `skills/` - Claude Code skill 소스. 각 `<skill-name>/` 가 `~/.claude/skills/<skill-name>/` 로 설치됨
- `commands/` - slash command 소스. 각 `<command-name>.md` 가 `~/.claude/commands/<command-name>.md` 로 설치됨
- `hooks/` - hook 조각 파일. `~/.claude/settings.json` 의 `hooks` 섹션으로 병합됨
- `permissions/` - permissions 조각 파일. `~/.claude/settings.json` 의 `permissions` 섹션으로 병합됨
- `docs/` - 설계·참고 문서

## 설치 구조 (모듈 임포트)

```
~/.claude/
├── CLAUDE.md                             사용자 파일. @claude-setting.md 한 줄만 보장.
├── claude-setting.md                     저장소 전담. 임포트 집약 파일.
└── imports/
    └── claude-setting/
        ├── common.md                     원본 복사
        ├── coding-guidelines.md          원본 복사
        ├── conventions/<lang>.md         원본 복사
        ├── platforms/<os>.md             선택된 플랫폼 파일 원본 복사
        ├── tools/<tool>.md               {{에디션}} 치환 완료본
        └── environment.md                장비별 환경 탐지 결과 (동적 생성, 저장소 원본 없음)
```

- `~/.claude/CLAUDE.md` 는 사용자 소유다. 저장소는 `@claude-setting.md` 임포트 라인 한 줄만 보장하고, 그 외 어떤 줄도 추가·제거·수정하지 않는다.
- 저장소가 관리하는 파일은 `claude-setting.md` 와 `imports/claude-setting/**` 다.
- `environment.md` 는 장비별 동적 생성물이다. 저장소에 원본이 없으며, 재설치 시 덮어쓰지 않고 유지한다.

## 설치 절차

사용자가 환경 설정을 요청하면 다음을 수행한다. (이 저장소 디렉토리에서 실행)

### 1단계: OS 확인
- 현재 시스템의 OS 를 확인하고 해당하는 `platforms/<os>.md` 를 선택한다.

### 2단계: 개발 도구 탐지 (최초 설치 시 / Windows 전용)
- `tools/` 의 각 도구 파일 "경로 탐지" 섹션에 따라 설치 여부·에디션·경로를 확인한다(기본 경로 → vswhere 순).
- 탐지된 값으로 `{{에디션}}` 등 플레이스홀더를 치환한다.
- 여러 버전이 발견되면 어느 것을 쓸지 사용자에게 확인한다.
- 이미 설치된 환경의 일반 업데이트에서는 재탐지하지 않는다.

### 3단계: 환경 탐지 (최초 설치 시)
- `platforms/<os>.md` 의 "환경 탐지 대상" 섹션에 따라 python/node/셸 등의 설치 여부·호출명·버전을 탐지한다.
- 탐지 결과로 `~/.claude/imports/claude-setting/environment.md` 를 생성한다(포맷은 하단 참조).
- `environment.md` 가 이미 존재하면 건드리지 않는다. 재탐지는 "환경 탐지 갱신 절차" 로만 수행한다.
- 환경 탐지의 핵심 목적은 "사용 가능한 명령/툴 목록"을 항상 로드하여, 없는 도구를 사용하려는 시도를 막는 것이다.

### 4단계: CLAUDE.md 모듈 설치 (덮어쓰기)
- 다음 파일을 `~/.claude/imports/claude-setting/` 아래로 복사한다(기존 파일은 덮어쓴다):
  - `common.md`, `coding-guidelines.md`, 선택된 `conventions/<lang>.md`, 선택된 `platforms/<os>.md` → 원본 그대로 복사
  - 선택된 `tools/<tool>.md` → `{{에디션}}` 치환 후 복사
- `~/.claude/claude-setting.md` 를 하단 "claude-setting.md 형식" 으로 재작성한다.
- `environment.md` 는 3단계 규칙을 따른다(없으면 생성, 있으면 유지).
- `~/.claude/CLAUDE.md` 에 `@claude-setting.md` 임포트 라인이 없으면 파일 최상단에 삽입한다. 있으면 그대로 둔다. 그 외 내용은 건드리지 않는다.

### 5단계: Skills/Commands 설치 (덮어쓰기)
- `skills/<name>/` 디렉토리 전체를 `~/.claude/skills/<name>/` 로 복사한다.
- `commands/<name>.md` 를 `~/.claude/commands/<name>.md` 로 복사한다.
- 각 디렉토리 **루트의 `README.md`** 는 제외한다. `skills/<name>/` **하위** 파일은 이름과 무관하게 모두 설치 대상이다.

### 6단계: Hooks 설치 (병합)
- `hooks/<name>.json` 조각을 `~/.claude/settings.json` 의 `hooks[event]` 배열에 병합한다(하단 "조각 파일 포맷" 참조).
- 동일 `event` + `matcher` 블록이 있으면 그 블록의 `hooks` 를 소스 값으로 교체하고, 없으면 추가한다.
- 사용자가 작성한 다른 hook 블록·다른 event 설정은 건드리지 않는다.
- `hooks/README.md` 는 제외한다.

### 7단계: Permissions 설치 (병합)
- `permissions/<name>.json` 의 `allow`/`deny`/`ask` 항목을 `~/.claude/settings.json` 의 동일 버킷 배열에 추가한다.
- 권한 문자열의 `{{홈}}` 플레이스홀더는 실제 홈 디렉토리 절대 경로로 치환한다.
- 이미 존재하는 항목은 중복 추가하지 않는다. 사용자가 추가한 다른 항목·순서는 건드리지 않는다.
- `permissions/README.md` 는 제외한다.

### 8단계: 환경 변수 적용
- `~/.claude/settings.json` 의 `env` 섹션에 아래 값을 병합한다(키가 있으면 덮어쓴다).
  ```json
  {
    "env": {
      "DISABLE_ERROR_REPORTING": "1",
      "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
    }
  }
  ```
- 다른 `env` 항목은 건드리지 않는다.

### 9단계: 결과 보고
- 적용된 설정 요약을 보고한다(OS, 탐지된 도구·환경 툴, 설치/갱신된 skill·command·hook·permission 개수 등).

## 업데이트 절차

사용자가 설정 최신화/업데이트를 요청하면(예: "업데이트해줘", "최신 설정으로 맞춰줘"):

- 설치 절차의 4~8단계를 동일하게 수행한다(저장소 소스를 `~/.claude` 로 덮어쓰기/병합).
- OS·도구·환경 재탐지는 하지 않는다. `environment.md` 는 유지한다.
- 저장소에 새 `conventions/<lang>.md` 등이 추가되었으면 함께 설치한다. 새 OS/도구 추가는 사용자에게 확인한다.
- 저장소에서 삭제된 항목이 `~/.claude` 에 남아 있어도 자동 제거하지 않는다. 보고만 하고, 제거는 사용자 요청 시에만 수행한다.
- 변경 내용을 요약 보고한다. 변경이 없으면 "이미 최신 상태" 임을 안내한다.

## 환경 탐지 갱신 절차

"툴 상황 업데이트해줘", "환경 재탐지" 등 **환경 정보만** 갱신하도록 명시적으로 요청할 때만 수행한다. Skills/Commands/Hooks/Permissions 는 건드리지 않는다.

1. 현재 OS 를 확인하여 `platforms/<os>.md` 를 선택한다.
2. "환경 탐지 대상" 섹션의 탐지 명령을 실행하여 각 툴의 설치 여부·호출명·버전을 수집한다. 명령 미발견(`command not found`, `is not recognized` 등)은 조용히 "사용 불가" 로 분류한다. Windows 에서 네이티브 탐지 실패 시 WSL 탐지를 시도한다(미설치 시 건너뜀).
3. `~/.claude/imports/claude-setting/environment.md` 를 전체 재작성한다(사용자 편집본이어도 덮어쓰며, 덮어쓰기 전 "재작성됨" 을 고지한다).
4. 새로 발견/사라진 툴, 버전 변경을 요약 보고한다.

## 외부 스킬 설치 절차

사용자가 "grillme 설치해줘", "grill-me 설치", "grill me 스킬 깔아줘" 등으로 특정 외부 스킬 설치를 요청하면 본 절차를 수행한다.

- 이 절차는 claude-setting 저장소가 관리하는 스킬(저장소 `skills/`)이 아니라, **외부 GitHub 저장소에서 직접 파일을 받아** `~/.claude/skills/` 에 설치하는 별도 경로다.
- **저장소 관리 대상이 아니다.** 1회성 설치이며, 충돌·삭제 추적을 하지 않는다. 최신 버전이 필요하면 사용자가 다시 설치를 요청할 때 소스에서 새로 받아 덮어쓴다.
- 이 환경에 Node.js 가 없으므로 일반적으로 안내되는 `npx skills add ...` 방식은 사용하지 않는다. GitHub raw URL 에서 파일을 직접 받아 기록한다(WebFetch).

### 설치 가능 스킬 목록

| 트리거 예시 | 스킬명 | 파일 | 소스 (GitHub raw) | 설치 대상 |
|---|---|---|---|---|
| "grillme 설치", "grill-me 설치", "grill me 깔아줘" | `grill-me` | `SKILL.md` | `https://raw.githubusercontent.com/mattpocock/skills/main/skills/productivity/grill-me/SKILL.md` | `~/.claude/skills/grill-me/SKILL.md` |

- 목록에 없는 스킬을 요청하면 사용자에게 "등록되지 않은 스킬" 임을 알리고, 소스 URL 을 함께 받으면 설치할 수 있음을 안내한다.
- 스킬이 여러 파일로 구성된 경우 각 파일을 같은 방식으로 받아 동일한 상대 경로로 기록한다(grill-me 는 `SKILL.md` 단일 파일).

### 절차

1. 요청을 위 목록과 대조하여 설치할 스킬을 식별한다.
2. 대상 디렉토리 `~/.claude/skills/<skill-name>/` 가 이미 존재하면, 덮어쓸지 사용자에게 확인한다(이전 설치본 또는 사용자 수정본일 수 있음). "최신으로 다시 설치" 처럼 갱신 의도가 명시되면 확인 없이 덮어쓴다.
3. 소스 raw URL 에서 파일 내용을 받아온다(WebFetch).
4. 받은 내용을 `~/.claude/skills/<skill-name>/<파일>` 에 그대로 기록한다(줄바꿈 변환 없음).
5. 설치 완료를 보고한다(스킬명, 설치 경로, 소스 URL).

## claude-setting.md 형식

저장소가 전체를 재작성한다.

```markdown
# claude-setting 임포트 집약 파일

이 파일은 `claude-setting` 저장소에 의해 자동 생성됩니다. 직접 편집하지 마십시오.
사용자 고유 지침은 `~/.claude/CLAUDE.md` 에 작성하십시오.

@imports/claude-setting/common.md
@imports/claude-setting/coding-guidelines.md
@imports/claude-setting/environment.md
```

- 임포트 순서: `common` → `coding-guidelines` → `environment`.
- `conventions/<lang>.md` 는 집약 파일에 포함하지 않는다. `coding-guidelines.md` 의 "언어별 코드 규칙" 섹션에서 작업 시 직접 참조한다.
- `platforms/<os>.md`, `tools/<tool>.md` 는 집약 파일에 포함하지 않는다. 환경 탐지·빌드 작업 시 직접 참조한다.
- `environment.md` 는 동적 생성물이지만 집약 파일에는 고정 라인으로 포함한다. 파일이 없으면 경고가 날 수 있으므로 설치 시점에 반드시 함께 생성한다.

## 조각 파일 포맷

### hooks/<name>.json
```json
{
  "event": "PreToolUse",
  "matcher": "WebSearch",
  "hooks": [
    { "type": "command", "command": "...", "timeout": 5 }
  ]
}
```
- `event`: `PreToolUse`, `PostToolUse`, `SessionStart`, `Stop` 등 hook 이벤트명.
- `matcher`: 해당 이벤트 배열 안에서 반응할 대상 패턴.
- `hooks`: `settings.json` 내 동일 위치에 들어갈 실행자 배열.
- 동일 `event` + `matcher` 조합의 파일은 저장소 내 하나만 존재해야 한다.

### permissions/<name>.json
```json
{
  "allow": ["WebFetch", "Bash(git log *)"],
  "deny": [],
  "ask": []
}
```
- `allow`/`deny`/`ask` 모두 선택 사항(생략·빈 배열 가능).
- 각 문자열은 Claude Code permission 규칙 문법을 따른다. `{{홈}}` 은 설치 시 홈 경로로 치환한다.
- 같은 권한 문자열을 여러 조각 파일이 중복 선언하지 않는다.

## environment.md 포맷

```markdown
# 환경 탐지 결과

본 문서는 현재 장비의 환경 탐지 결과입니다. "툴 상황 업데이트해줘" 요청 시 재작성됩니다.

## 사용 가능 툴

- **Python**: `python3` — Python 3.12.3 (`/usr/bin/python3`)
- **.NET SDK**: `dotnet` — 10.0.104

## 사용 불가 툴

- **Node.js**: 설치되어 있지 않음

## WSL 경유 (Windows 전용)

- **<툴명>**: `wsl -- <tool>` — <버전>

## 포인터 참조

- **MSBuild**: VS 설정 참조 (`tools/vs<버전>.md`)
```

- "사용 가능 툴" 에는 호출 명령과 버전을 함께 기록한다.
- "사용 불가 툴" 이 없으면 "(없음)" 으로 명시한다(섹션 생략 금지).
- "WSL 경유" 는 Windows 가 아니거나 해당 항목이 없으면 생략한다. "포인터 참조" 도 항목이 없으면 생략한다.
- 호출명 후보가 여럿인 툴(`python`/`python3`, `pwsh`/`powershell`)은 먼저 발견된 호출명을 대표로 기록하고, 다른 호출명이 함께 있으면 비고에 표기한다.

## 실행 환경 고려사항

- 경로 구분자는 각 OS 에 맞게 처리한다. `@` 임포트 라인은 POSIX 슬래시(`@imports/claude-setting/...`)로 작성한다(Windows 에서도 해석됨).
- 파일 복사 시 줄바꿈 변환을 하지 않고 소스 바이트 그대로 복사한다(`tools/*.md` 는 플레이스홀더 치환 뒤 바이트).
- `settings.json` 을 쓸 때 기존 들여쓰기 스타일(보통 2칸)과 키 순서를 가능한 한 유지한다.
