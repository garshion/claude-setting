# Claude Code 확장 메커니즘 정리

Claude Code를 확장하는 네 가지 주요 메커니즘(Skills, Slash Commands, Hooks, Rules/CLAUDE.md)의 동작·등록 방식을 정리한 문서.
본 저장소에 기능을 하나씩 적용하기 위한 참고 자료.

> **주의**: 일부 세부 사항(예: `.claude/rules/` 의 `paths` frontmatter 기반 경로 활성화)은 공식 기능 여부가 불확실한 부분이 있음. 적용 전 공식 문서 재확인 권장.

공식 문서:
- Skills: https://code.claude.com/docs/en/skills.md
- Hooks: https://code.claude.com/docs/en/hooks.md
- Memory & CLAUDE.md: https://code.claude.com/docs/en/memory.md

---

## 1. Skills — 주문형 지침 번들

### 개념
- CLAUDE.md: 세션 시작 시 **항상** 로드되는 상시 지침.
- Skill: **필요할 때만** 로드되는 지침. 컨텍스트 절약이 목적.

### 디렉토리 구조
```
~/.claude/skills/my-skill/          (전역, 모든 프로젝트에 적용)
.claude/skills/my-skill/            (프로젝트, 깃 공유)
├── SKILL.md                        필수. frontmatter + 지침 본문
├── reference.md                    선택. 상세 참고 자료
├── template.md                     선택. 템플릿
├── examples/                       선택. 예시
│   └── sample.md
└── scripts/                        선택. skill이 호출할 스크립트
    └── validate.sh
```

`SKILL.md`만 필수. 나머지는 본문에서 참조하면 필요 시 Claude가 로드.

### 계층과 우선순위

| 우선순위 | 위치 | 범위 |
|---|---|---|
| 1 | 엔터프라이즈 관리 정책 | 조직 전체 |
| 2 | `~/.claude/skills/<name>/SKILL.md` | 모든 프로젝트(개인) |
| 3 | `.claude/skills/<name>/SKILL.md` | 현재 프로젝트(깃 공유) |
| 4 | `<plugin>/skills/<name>/SKILL.md` | 활성 플러그인 범위 |

동일 이름이면 상위 우선순위가 이김. 플러그인 skill은 `plugin-name:skill-name` 네임스페이스 사용.

### SKILL.md frontmatter 주요 필드

```yaml
---
name: fix-issue                      # 기본값: 디렉토리명
description: Fix a GitHub issue      # 자동 호출 키워드 매칭 대상
when_to_use: "GitHub issue number"   # 추가 트리거 힌트
argument-hint: "[issue-number]"      # 자동완성 힌트
disable-model-invocation: true       # true: 사용자만 수동 호출 가능
user-invocable: false                # false: 메뉴에서 숨김
allowed-tools: Bash(git *) Read      # skill 활성 중 사전 승인 도구
model: claude-opus                   # skill 활성 중 모델 오버라이드
effort: high                         # 노력 수준 오버라이드
context: fork                        # fork: subagent에서 격리 실행
agent: Explore                       # context: fork일 때 agent 타입
paths:                               # glob 매칭 파일 작업 시에만 로드
  - "src/api/**/*.ts"
shell: bash                          # bash | powershell (기본 bash)
---
```

### 본문 내 동적 대체(string substitution)

| 변수 | 의미 |
|---|---|
| `$ARGUMENTS` | 호출 시 전달된 모든 인자 |
| `$0`, `$1`, ... / `$ARGUMENTS[N]` | N번째 인자(0-based) |
| `${CLAUDE_SESSION_ID}` | 현재 세션 ID |
| `${CLAUDE_SKILL_DIR}` | 해당 SKILL.md의 디렉토리 |

### 본문 내 셸 명령 실행(컨텍스트 주입)

렌더링 전 실행되고 결과가 본문에 삽입됨:

````markdown
PR 정보:
- Diff: !`gh pr diff`
- Files: !`gh pr diff --name-only`

여러 줄:
```!
node --version
npm --version
git status --short
```
````

### 인식/로드 메커니즘
- 세션 시작 시 `~/.claude/skills/`, `.claude/skills/` 스캔.
- **description만** 상시 컨텍스트에 로드(Claude가 호출 여부 판단용). 본문은 호출 시에만 로드.
- 한 번 호출된 skill은 세션 내내 유지(재로드 없음).
- 파일 추가/편집/삭제가 현재 세션에 실시간 반영. 단 새 디렉토리 생성은 재시작 필요.
- 하위 디렉토리의 `.claude/skills/` 도 자동 발견(모노레포 지원).

### 호출 방식
- **수동**: `/skill-name 인자` 형태로 사용자가 호출.
- **자동**: description·when_to_use 키워드 매칭 시 Claude가 호출. `disable-model-invocation: true` 이면 금지.
- **경로 기반 활성화**: `paths` frontmatter가 매칭되는 파일을 Claude가 작업할 때만 로드.

---

## 2. Slash Commands — 사용자 제어 명령

### 개념
`/명령` 형태로 사용자가 트리거하는 커스텀 명령.
최신 Claude Code에서는 skill로 통합되는 추세지만 command 파일 방식도 여전히 유효.

### 위치
| 위치 | 범위 |
|---|---|
| `~/.claude/commands/<name>.md` | 전역 |
| `.claude/commands/<name>.md` | 프로젝트 |
| `.claude/skills/<name>/SKILL.md` (`disable-model-invocation: true`) | skill로 command화 |

### 파일 형식

```markdown
---
description: Deploy the application
argument-hint: "[env]"
allowed-tools: Bash(npm *) Read
disable-model-invocation: true
---

배포 절차:
1. 테스트 실행
2. 빌드
3. $ARGUMENTS 환경으로 배포
```

### 인자 처리
- `/fix-issue 123` → `$ARGUMENTS = "123"`
- 공백 분리 위치 인자: `$0`, `$1`, `$2`
- 예: `/migrate-component SearchBar React Vue` → `$0=SearchBar, $1=React, $2=Vue`
- 본문에 `$ARGUMENTS` 없으면 자동으로 끝에 `ARGUMENTS: <value>` 추가됨.

### 로드
- 세션 시작 시 일괄 발견.
- 중복 시: skill > command, 개인 > 프로젝트 우선순위.

---

## 3. Hooks — 이벤트 기반 자동화

### 개념
Claude Code 라이프사이클 특정 시점에 자동 실행되는 커스텀 명령/HTTP 호출/프롬프트/에이전트.
Tool 실행 전 검증, 세션 시작 초기화, 사용자 입력 필터링 등.

### 주요 이벤트

| Hook | 시점 | 용도 예 |
|---|---|---|
| `SessionStart` | 세션 시작 | 환경 변수 설정, 초기화 |
| `SessionEnd` | 세션 종료 | 정리, 로깅 |
| `UserPromptSubmit` | 사용자 프롬프트 제출 시 | 입력 검증, 컨텍스트 추가 |
| `Stop` | 턴 완료 시 | 정리, 상태 저장 |
| `StopFailure` | 턴 실패 시 | 에러 처리 |
| `PreToolUse` | Tool 실행 전 | Permission 검증/수정/차단 |
| `PostToolUse` | Tool 실행 후 | 출력 검증, 로깅 |
| `PostToolUseFailure` | Tool 실행 실패 시 | 에러 처리 |
| `PermissionRequest` | Permission 다이얼로그 시 | 자동 승인/거부 |
| `PermissionDenied` | Permission 거부 시 | 로깅, 알림 |
| `Notification` | 알림 생성 시 | 알림 필터링 |
| `SubagentStart` / `SubagentStop` | 서브에이전트 시작/종료 | 초기화/정리 |
| `FileChanged` | 파일 변경 감지 시 | 포맷 검증 |
| `CwdChanged` | 작업 디렉토리 변경 | 상황별 설정 |
| `ConfigChange` | 설정 변경 | 검증 |
| `InstructionsLoaded` | CLAUDE.md 로드 시 | 로깅 |
| 기타 | `PreCompact` / `PostCompact`, `TaskCreated` 등 | 커스텀 자동화 |

### 정의 위치
| 파일 | 범위 |
|---|---|
| `~/.claude/settings.json` | 전역 |
| `.claude/settings.json` | 프로젝트(깃 공유) |
| `.claude/settings.local.json` | 프로젝트 로컬(gitignore) |
| 플러그인 `hooks/hooks.json` | 플러그인 범위 |
| Skill/Agent frontmatter의 `hooks:` | 해당 skill/agent 활성 중만 |

### 구조 예시

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "if": "Bash(rm *)",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/block-rm.sh",
            "timeout": 5
          }
        ]
      },
      {
        "matcher": "WebSearch",
        "hooks": [
          {
            "type": "command",
            "command": "echo '{\"hookSpecificOutput\":{\"hookEventName\":\"PreToolUse\",\"permissionDecision\":\"allow\"}}'",
            "timeout": 5
          }
        ]
      }
    ]
  }
}
```

### Matcher 패턴

| 값 | 해석 |
|---|---|
| `"*"`, `""`, 생략 | 모두 매칭 |
| 문자/숫자/`_`/`\|`만 사용 | 정확 일치 또는 `A\|B` 목록 |
| 기타 특수문자 포함 | 정규식 |
| MCP 도구 | `mcp__<server>__<tool>` 패턴, 예: `mcp__memory__.*` |

### Hook 타입

**command**: 스크립트/셸 명령 실행
- stdin으로 JSON 입력 수신
- stdout으로 JSON 출력 반환(선택)
- exit code: `0` 정상, `2` 차단(stderr이 Claude에 전달), 그 외 비차단 에러

**http**: HTTP 엔드포인트 호출
```json
{
  "type": "http",
  "url": "http://localhost:8080/hooks/pre-tool-use",
  "timeout": 30,
  "headers": { "Authorization": "Bearer $MY_TOKEN" },
  "allowedEnvVars": ["MY_TOKEN"]
}
```

**prompt**: Claude에게 프롬프트 실행
```json
{ "type": "prompt", "prompt": "사용자 프롬프트를 검증하세요.", "timeout": 60 }
```

**agent**: 서브에이전트 실행
```json
{ "type": "agent", "agent": "Explore", "timeout": 600 }
```

### Command hook 입출력

**입력(stdin, JSON)**:
```json
{
  "session_id": "abc123",
  "transcript_path": "/path/transcript.jsonl",
  "cwd": "/current/dir",
  "permission_mode": "default",
  "hook_event_name": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": { "command": "rm -rf /" }
}
```

**출력(stdout, JSON)**:
```json
{
  "continue": true,
  "stopReason": "...",
  "suppressOutput": false,
  "systemMessage": "사용자에게 표시할 경고",
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "allow|deny|ask|defer",
    "permissionDecisionReason": "이유",
    "updatedInput": { "command": "safe-version" }
  }
}
```

> `hookSpecificOutput.hookEventName` 필드가 없으면 hook이 동작하지 않음. 반드시 포함.

### 환경 변수
```
$CLAUDE_PROJECT_DIR       프로젝트 루트
${CLAUDE_PLUGIN_ROOT}     플러그인 디렉토리
${CLAUDE_PLUGIN_DATA}     플러그인 지속 데이터
```

### 기타
- 설정 변경이 현재 세션에서 감지됨(재시작 불필요).
- `/hooks` 명령으로 현재 설정된 hook 목록 조회.

---

## 4. Rules / CLAUDE.md 시스템

### 공식 기능 여부
- **지침 주입의 공식 메커니즘은 CLAUDE.md**.
- `.claude/rules/` 는 일부 문서/관행에서 언급되나, `paths` frontmatter 기반 경로 활성화 등 세부 기능이 공식 스펙인지 별도 검증 필요.

### CLAUDE.md 로드 계층(세션 시작 시 연결)

| 순서 | 위치 | 범위 |
|---|---|---|
| 1 | 관리 정책 CLAUDE.md(OS별 경로 상이) | 조직 전체 |
| 2 | `./CLAUDE.md` 또는 `./.claude/CLAUDE.md` | 프로젝트 루트 |
| 3 | `../CLAUDE.md`, `../../CLAUDE.md` ... | 상위 디렉토리(모노레포 대응) |
| 4 | `~/.claude/CLAUDE.md` | 사용자 전역 |
| 5 | `./CLAUDE.local.md` | 프로젝트 로컬(gitignore 권장) |

하위 디렉토리의 `CLAUDE.md` 도 해당 디렉토리 파일을 읽을 때 on-demand 로드됨.

### 형식

```markdown
# 프로젝트 구조
- `src/` — 소스
- `tests/` — 테스트

# 코드 규칙
- 2칸 들여쓰기
- TypeScript 사용

<!-- 주석은 context로 전달되지 않음(휴먼 메모용) -->
```

### 파일 임포트(`@` 문법)

```markdown
커밋 규칙은 @docs/git-instructions.md 를 따릅니다.
@.claude/rules/coding-standards.md
```

- 상대/절대 경로 모두 가능.
- 최대 5단계 재귀 임포트.

### `.claude/rules/` (비공식 관행 포함)

```
.claude/rules/
├── code-style.md
├── api-design.md
├── frontend/react.md
└── backend/rest.md
```

경로 기반 활성화를 위해 frontmatter 사용 사례가 보고됨(공식 여부 재확인 필요):

```markdown
---
paths:
  - "src/api/**/*.ts"
---
# API 설계 규칙
- 입력 검증 필수
- 표준 에러 포맷 사용
```

### CLAUDE.md 제외(모노레포)

```json
// .claude/settings.local.json
{
  "claudeMdExcludes": [
    "**/monorepo/CLAUDE.md",
    "/home/user/monorepo/other-team/.claude/rules/**"
  ]
}
```

글롭 패턴·절대 경로 매칭. 관리 정책 CLAUDE.md 는 제외 불가.

### CLAUDE.md vs settings.json

| 항목 | CLAUDE.md | settings.json `permissions` |
|---|---|---|
| 강제성 | Claude에게 주는 **제안** | 하네스가 **강제** |
| 용도 | 코드 스타일, 워크플로우, 아키텍처 | Tool 차단, 샌드박스, 환경 변수 |
| 위반 가능 | Claude 판단에 따라 위반 가능 | 클라이언트 수준에서 차단 |

### CLAUDE.md vs Auto Memory

| 항목 | CLAUDE.md | Auto Memory |
|---|---|---|
| 작성자 | 사용자 | Claude |
| 내용 | 지침·규칙 | 학습·패턴 |
| 저장 위치 | `./CLAUDE.md` 등 | `~/.claude/projects/<project>/memory/` |
| 로드 시점 | 세션 시작 | 세션 시작(200줄 또는 25KB 제한) |

---

## 전체 요약

| 기능 | 파일 | 위치 | 로드 시점 | 자동 호출 | 공유 |
|---|---|---|---|---|---|
| Skill | `SKILL.md` | `~/.claude/skills/<name>/` 또는 `.claude/skills/<name>/` | description은 세션 시작, 본문은 호출 시 | 기본 yes(disable 가능) | 깃 가능 |
| Slash Command | `<name>.md` | `~/.claude/commands/` 또는 `.claude/commands/` | 세션 시작 | 수동만 | 깃 가능 |
| Hook | JSON | `~/.claude/settings.json` 또는 `.claude/settings.json` | 이벤트 발생 시 | 자동 | 깃 가능(local 제외) |
| CLAUDE.md | `.md` | 프로젝트/상위/사용자 계층 | 세션 시작 | (컨텍스트 제안) | 깃 가능 |
| Rules | `.md` | `.claude/rules/` | 세션 시작(경로 기반은 on-demand) | (컨텍스트 제안) | 깃 가능 |

---

## 본 저장소에의 응용 포인트

1. **공통 지침은 CLAUDE.md + `@임포트`로 모듈화**
   현재 `common.md` / `platforms/*.md` / `tools/*.md` 조합 방식을 CLAUDE.md 임포트로 연결 가능.

2. **반복 절차는 skill 또는 slash command로 패키징**
   설치/업데이트 절차를 `/update-settings` 류 명령으로 호출 가능하게.

3. **검증·자동화는 hook으로**
   설정 저장 시 JSON 유효성 검사, 특정 명령 자동 승인/거부 등.
   현재 `settings.json`의 WebSearch 자동 승인이 이 패턴의 실제 적용 예.

4. **강제해야 할 것은 settings.json permissions로**
   제안 수준이 아닌 차단이 필요한 경우 사용.
