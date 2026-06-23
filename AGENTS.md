# Codex 글로벌 설정 관리 저장소

이 저장소는 Codex의 글로벌 `AGENTS.md` 지침과 skill을 관리한다.
설치/업데이트는 **이 저장소 디렉터리에서 직접 요청**한다(저장소 경로를 따로 기록하지 않는다).

설치/업데이트의 기본 동작은 **단순 덮어쓰기**다. 매니페스트·해시·충돌 판정을 사용하지 않는다.
원칙: 설치본(`~/.codex`, `~/.agents`)을 직접 편집하지 않는다. 고칠 내용은 저장소 소스를 고치고 재설치한다.

## Codex 적용 범위

- `common.md` - `~/.codex/AGENTS.md`의 저장소 관리 블록에 포함되는 기본 지침
- `coding-guidelines.md` - 코딩 작업 전에 읽는 공통 가이드라인
- `conventions/` - 언어별 코드 규칙 (cpp.md, csharp.md 등. 온디맨드)
- `platforms/` - OS별 환경 탐지 대상 (windows.md, mac.md, linux.md. 온디맨드)
- `tools/` - 개발 도구별 설정 (vs2022.md, vs2026.md 등. `{{에디션}}` 치환. 온디맨드)
- `skills/` - Codex에서 사용할 skill 소스. 각 `<skill-name>/`를 `~/.agents/skills/<skill-name>/`로 설치

다음 항목은 Claude Code 전용 형식이므로 Codex 설치에서는 다루지 않는다.

- `commands/` - Codex의 custom prompt는 폐기 예정이며, 재사용 워크플로는 skill을 사용한다.
- `hooks/`, `permissions/` - 저장소 조각 파일이 Claude Code의 `settings.json` 형식이다.
- Claude 전용 환경 변수 `DISABLE_ERROR_REPORTING`, `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`

## 설치 구조

```text
~/.codex/
├── AGENTS.md                              사용자 파일. 저장소 관리 블록만 갱신.
└── instructions/
    └── claude-setting/
        ├── coding-guidelines.md           Codex 경로로 참조를 치환한 복사본
        ├── conventions/<lang>.md          원본 복사
        ├── platforms/<os>.md              선택된 플랫폼 파일 원본 복사
        ├── tools/<tool>.md                 {{에디션}} 치환 완료본
        └── environment.md                 장비별 환경 탐지 결과 (동적 생성)

~/.agents/
└── skills/
    └── <skill-name>/                      저장소 skill 디렉터리 복사본
```

- `~/.codex/AGENTS.md`는 사용자 소유다. 저장소는 하단의 시작/끝 표식 사이만 추가·교체하고, 그 밖의 내용은 건드리지 않는다.
- `environment.md`는 장비별 동적 생성물이다. 저장소에 원본이 없으며 일반 업데이트 시 유지한다.
- Codex는 실행을 시작할 때 글로벌 `~/.codex/AGENTS.md`를 읽는다. 설치 또는 갱신 후에는 새 세션을 시작해야 반영된다.

## 설치 절차

사용자가 Codex 환경 설정을 요청하면 다음을 수행한다. 이 저장소 디렉터리에서 실행한다.

### 1단계: OS 확인

- 현재 시스템의 OS를 확인하고 해당하는 `platforms/<os>.md`를 선택한다.

### 2단계: 개발 도구 탐지 (최초 설치 시 / Windows 전용)

- `tools/`의 각 도구 파일에 있는 "경로 탐지" 절차에 따라 설치 여부·에디션·경로를 확인한다(기본 경로 → vswhere 순).
- 탐지된 값으로 `{{에디션}}` 등의 플레이스홀더를 치환한다.
- 여러 버전이 발견되면 어느 것을 쓸지 사용자에게 확인한다.
- 이미 설치된 환경의 일반 업데이트에서는 재탐지하지 않는다.

### 3단계: 환경 탐지 (최초 설치 시)

- `platforms/<os>.md`의 "환경 탐지 대상"에 따라 Python, Node.js, 셸 등의 설치 여부·호출명·버전을 탐지한다.
- 탐지 결과로 `~/.codex/instructions/claude-setting/environment.md`를 생성한다(포맷은 하단 참조).
- `environment.md`가 이미 존재하면 건드리지 않는다. 재탐지는 "환경 탐지 갱신 절차"로만 수행한다.

### 4단계: 지침 모듈 설치 (덮어쓰기)

- `~/.codex/instructions/claude-setting/` 디렉터리를 만든다.
- 다음 파일을 해당 디렉터리 아래로 복사한다.
  - 선택된 `conventions/<lang>.md`, `platforms/<os>.md`는 원본 그대로 복사한다.
  - `coding-guidelines.md`는 `~/.claude/imports/claude-setting/` 참조를 `~/.codex/instructions/claude-setting/`으로 치환해 복사한다.
  - 선택된 `tools/<tool>.md`는 `{{에디션}}` 등의 플레이스홀더를 치환해 복사한다.
- `environment.md`는 3단계 규칙을 따른다(없으면 생성, 있으면 유지).
- `~/.codex/AGENTS.md`에 하단 "관리 블록 형식"의 시작/끝 표식이 모두 있으면 블록 전체를 교체한다.
- 표식이 없으면 파일 끝에 빈 줄을 둔 뒤 관리 블록을 추가한다. 파일이 없으면 관리 블록만 포함해 생성한다.
- 표식이 하나만 있거나 순서가 잘못되었으면 임의로 수정하지 말고 사용자에게 확인한다.

### 5단계: Skills 설치 (덮어쓰기)

- `skills/<name>/` 디렉터리 전체를 `~/.agents/skills/<name>/`로 복사한다.
- `skills/README.md`는 제외한다. 각 skill 디렉터리 내부 파일은 이름과 무관하게 모두 설치한다.
- Codex는 `SKILL.md`의 `name`과 `description`을 기준으로 skill을 발견한다. Claude Code 전용 front matter가 있더라도 설치 시 임의로 제거하거나 변환하지 않는다.

### 6단계: 결과 보고

- 적용된 설정을 요약 보고한다(OS, 탐지된 도구·환경 툴, 설치/갱신된 지침 파일과 skill 개수, 제외된 Claude 전용 항목).
- 변경 사항을 사용하려면 Codex 새 세션이 필요함을 안내한다.

## 업데이트 절차

사용자가 Codex 설정 최신화/업데이트를 요청하면 다음을 수행한다.

- 설치 절차의 4~5단계를 동일하게 수행한다.
- OS·도구·환경은 재탐지하지 않고 기존 `environment.md`를 유지한다.
- 저장소에 새 `conventions/<lang>.md`가 추가되었으면 함께 설치한다. 새 OS/도구가 추가되었으면 사용자에게 확인한다.
- 저장소에서 삭제된 파일이나 skill이 설치본에 남아 있어도 자동 제거하지 않는다. 차이만 보고하고, 제거는 사용자 요청 시에만 수행한다.
- 변경이 없으면 이미 최신 상태임을 안내한다.

## 환경 탐지 갱신 절차

"툴 상황 업데이트해줘", "환경 재탐지" 등 환경 정보 갱신을 명시적으로 요청할 때만 수행한다. 다른 지침과 skill은 건드리지 않는다.

1. 현재 OS를 확인하여 `platforms/<os>.md`를 선택한다.
2. "환경 탐지 대상"의 명령을 실행해 각 툴의 설치 여부·호출명·버전을 수집한다. 명령 미발견은 조용히 "사용 불가"로 분류한다. Windows 네이티브 탐지 실패 시 WSL 탐지를 시도한다(WSL 미설치 시 건너뜀).
3. 기존 파일을 재작성한다고 먼저 고지한 뒤 `~/.codex/instructions/claude-setting/environment.md`를 전체 재작성한다.
4. 새로 발견되거나 사라진 툴과 버전 변경을 요약 보고한다.

## 외부 스킬 설치 절차

- 사용자가 외부 Codex skill 설치를 요청하면 우선 내장 `$skill-installer`를 사용한다.
- 설치 대상은 사용자 범위의 `~/.agents/skills/<skill-name>/`다.
- 이미 같은 이름의 skill이 있으면, 갱신 의도가 명시되지 않은 한 덮어쓰기 전에 사용자에게 확인한다.
- 이 저장소의 `skills/`에 없는 외부 skill은 저장소 관리 대상이 아니며, 이후 저장소 업데이트에서 변경하거나 삭제하지 않는다.

## 관리 블록 형식

`common.md`의 전체 내용을 `<common.md 내용>` 위치에 삽입한다. 표식 문구는 정확히 유지한다.

```markdown
<!-- claude-setting:codex:start -->
# claude-setting 관리 지침

이 블록은 `claude-setting` 저장소가 관리합니다. 직접 편집하지 마십시오.
사용자 고유 지침은 이 블록 밖에 작성하십시오.

<common.md 내용>

## 작업별 추가 지침

- 소스 코드 작성 및 관련 작업 전에는 `~/.codex/instructions/claude-setting/coding-guidelines.md`를 읽고 따를 것.
- C# 또는 C++ 작업 시에는 코딩 가이드라인이 가리키는 언어별 convention 파일을 읽고 따를 것.
- 빌드 도구 사용 전에는 `~/.codex/instructions/claude-setting/environment.md`의 포인터를 보고 필요한 tool 파일을 읽을 것.
<!-- claude-setting:codex:end -->
```

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

- "사용 가능 툴"에는 호출 명령과 버전을 함께 기록한다.
- "사용 불가 툴"이 없으면 "(없음)"으로 명시한다(섹션 생략 금지).
- "WSL 경유"는 Windows가 아니거나 해당 항목이 없으면 생략한다. "포인터 참조"도 항목이 없으면 생략한다.
- 호출명 후보가 여러 개인 툴은 먼저 발견된 호출명을 대표로 기록하고, 다른 호출명도 사용 가능하면 비고에 표기한다.

## 실행 환경 고려사항

- 경로 구분자는 각 OS에 맞게 처리한다.
- 원본 복사 대상은 줄바꿈을 변환하지 않고 소스 바이트 그대로 복사한다. 경로·플레이스홀더 치환 대상만 텍스트로 변환한다.
- 사용자 홈 밖에 쓰거나 기존 사용자 파일을 변경하기 전에 현재 세션의 권한·승인 정책을 따른다.
