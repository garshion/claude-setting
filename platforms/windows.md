# 환경 탐지 대상

저장소 `CLAUDE.md` 의 "환경 탐지 세부 규칙" 에 따라 `environment.md` 를 생성·갱신할 때 본 플랫폼에서 탐지하는 툴 목록이다.

## 탐지 대상

| 툴 | 네이티브 탐지 | WSL 탐지 | 비고 |
| --- | --- | --- | --- |
| Python | `where python` / `where python3` 후 해당 명령으로 `<cmd> --version` | `wsl -- command -v python3 && wsl -- python3 --version` | Windows 네이티브는 `python` 만 유효(공식 인스톨러는 `python3.exe` 를 생성하지 않음). `where python3` 결과가 `WindowsApps/python3.exe` 뿐이면 Store 스텁이므로 네이티브는 "사용 불가" 로 기록하고 `python3` 이 필요하면 WSL 경유로 기록 |
| Node.js | `where node` 후 `node --version` | `wsl -- command -v node && wsl -- node --version` | |
| PowerShell | `where pwsh` / `where powershell` 후 `<cmd> -NoProfile -Command "$PSVersionTable.PSVersion.ToString()"` | — | PS Core(`pwsh`) / Windows PS(`powershell`) 구분 기록 |
| .NET SDK | `where dotnet` 후 `dotnet --version` | `wsl -- command -v dotnet && wsl -- dotnet --version` | |

## MSBuild 포인터

- MSBuild 는 Visual Studio 설치본에 포함되므로 별도 탐지하지 않는다.
- `tools/vs*.md` 의 탐지 결과(에디션·경로)를 그대로 사용한다.
- `environment.md` 에는 `MSBuild: VS 설정 참조 (tools/vs<버전>.md)` 형식의 포인터만 기록한다.

## WSL 처리

- 네이티브 실행 파일이 발견되지 않은 경우에만 WSL 탐지를 시도한다.
- `where wsl` 이 실패하거나 WSL 배포판이 없으면 WSL 탐지를 건너뛴다.
- WSL 경유로만 사용 가능한 경우 `environment.md` 에 호출 방법을 `wsl -- <tool>` 형식으로 기록한다.

# 툴 설치 방법

사용자가 특정 툴의 설치를 요청하면 아래 지침에 따라 `winget` 으로 설치한다.

## 공통 사항

- 설치 후 **이미 열려 있는 셸 세션에서는 PATH 가 갱신되지 않는다.** 새 셸을 열어야 `where <tool>` 등으로 확인 가능함을 사용자에게 안내한다.
- **VSCode 등 IDE 의 PATH 상속 주의**: 창을 닫고 다시 열어도 트레이/백그라운드 프로세스가 남아 이전 환경변수를 자식 셸에 상속시킨다. 새 셸에서도 PATH 가 반영되지 않으면 **작업 관리자에서 `Code.exe`(또는 해당 IDE) 프로세스를 전부 종료한 뒤 재실행** 하도록 안내한다.
- `winget install` 은 기본 permission 에서 프롬프트가 발생할 수 있다. 프롬프트가 나오면 사용자 승인 후 진행한다.
- 설치 완료 후 "환경 탐지 갱신 절차" 를 수행하여 `environment.md` 를 최신화하도록 사용자에게 제안한다.

### winget 필수 플래그

비대화형 환경(본 에이전트 실행 포함)에서는 약관 프롬프트가 뜨면 입력 처리 에러(`0x8a150042 : Error reading input in prompt`, exit code 66 등)로 설치가 실패한다. 따라서 설치 명령에는 항상 다음 플래그를 함께 사용한다:

- `--source winget` — msstore 소스 자동 선택으로 인한 약관 프롬프트를 회피한다(의도한 공식 패키지를 명확히 지정).
- `--accept-source-agreements` — 소스 약관 자동 동의.
- `--accept-package-agreements` — 패키지 약관 자동 동의.
- `--id <패키지 ID>` — 이름 매칭 모호성 제거(추천).

## Python

- **기본 설치**: `winget install --id Python.Python.3.13 --source winget --accept-source-agreements --accept-package-agreements`
  - 파이썬은 공식 LTS 개념이 없으며 3.13 은 안정 버전으로 권장한다.
  - 용도가 단순 스크립팅(해시, JSON 처리 등)이라면 3.13 으로 충분하다.
- **특정 버전 요구**: `winget install --id Python.Python.3.<minor> --source winget --accept-source-agreements --accept-package-agreements` (예: 3.12, 3.14)
  - 특정 SDK 가 구버전을 요구하거나 병행 설치가 필요한 경우.
  - 여러 버전을 상시 오갈 필요가 생기면 `pyenv-win` 도입을 제안한다(본 저장소에서는 기본 채택하지 않음).
- **PATH 자동 등록**: 사용자 스코프에 `python.exe`, `<install>/Scripts/` 경로가 추가된다. winget/공식 인스톨러는 `WindowsApps` 보다 **앞** 에 삽입하므로 `where python` 결과도 실제 설치본이 먼저 나온다.
- **호출명은 `python` 만 유효**: 공식 인스톨러는 `python.exe` / `pythonw.exe` 만 배포하며 `python3.exe` 링크는 **생성하지 않는다**. 설치 후에도 `python3` 호출은 `WindowsApps/python3.exe`(Windows Store 스텁)가 가로채 설치 안내를 출력할 수 있으므로, Windows 네이티브에서는 `python` 호출명을 사용한다. `python3` 이 필요하면 WSL 경유.
- **Windows Store 버전은 권장하지 않음**: 설치 경로(`WindowsApps`)가 비표준이어서 일부 개발 툴과 호환성 이슈가 발생할 수 있다.

## Node.js

- **기본 설치 (LTS)**: `winget install --id OpenJS.NodeJS.LTS --source winget --accept-source-agreements --accept-package-agreements`
- **최신 Current 버전**: `winget install --id OpenJS.NodeJS --source winget --accept-source-agreements --accept-package-agreements`
  - Current 는 최신 기능이 먼저 들어오지만 수명이 짧다. 특별한 이유가 없으면 LTS 를 권장한다.
- **PATH 자동 등록**: 사용자 스코프에 `node.exe`, `npm.cmd` 경로가 추가된다.
- **Corepack**: Node 설치에 `corepack` 이 함께 포함되지만 기본 비활성 상태다. `yarn`/`pnpm` 사용 예정이면 `corepack enable` 을 안내한다.

# 주의사항
- **줄바꿈 정규화**: Write 도구가 LF로 파일을 쓰지만, `.gitattributes`에서 CRLF 강제하는 프로젝트가 있음. 파일 작업 완료 후 반드시 `git add --renormalize .` 실행할 것. 단, `git add`로 먼저 스테이징하면 renormalize가 효과 없으므로, 스테이징 없이 renormalize만 실행할 것.
