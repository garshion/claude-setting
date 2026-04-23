# macOS 설정

## 경로 탐지

### OS 정보
- macOS 버전: `sw_vers -productVersion`
- 빌드: `sw_vers -buildVersion`
- 커널: `uname -r`
- 아키텍처: `uname -m` (`arm64` = Apple Silicon, `x86_64` = Intel)
- 하드웨어 모델: `sysctl -n hw.model`

### Homebrew 탐지
- 설치 여부: `command -v brew`
- Prefix:
  - Apple Silicon: `/opt/homebrew`
  - Intel: `/usr/local`
- Homebrew 가 없으면 "사용 불가" 로 기록하고, 사용자가 툴 설치를 요청할 때 먼저 Homebrew 설치를 안내한다.

### IDE 탐지
| IDE | 탐지 명령 | 기본 탐지 |
|-----|-----------|-----------|
| VSCode | `command -v code && code --version \| head -1` | O |
| Cursor | `command -v cursor && cursor --version \| head -1` | |
| JetBrains IDEs | `command -v idea`, `command -v clion`, `command -v rider`, `command -v goland` | |
| Xcode | `xcode-select -p && xcodebuild -version \| head -1` | O |

- VSCode/Cursor/JetBrains IDE 의 CLI 런처는 앱에서 "Install 'code' command in PATH" 명령을 실행해야 설치된다. 미설치 시 "사용 불가" 로 기록한다.
- Xcode Command Line Tools 만 있고 Xcode 풀 설치가 없는 경우 `xcode-select -p` 는 `/Library/Developer/CommandLineTools` 를 반환한다. 이 경우 "Xcode CLT 만 설치" 로 기록한다.

### 런타임/컴파일러 탐지
| 도구 | 탐지 명령 | 기본 탐지 |
|------|-----------|-----------|
| Clang/Clang++ | `clang --version \| head -1` | O |
| Python | `python3 --version` | O |
| Node.js | `node --version` | O |
| .NET | `dotnet --version` | |
| Go | `go version` | |
| Rust | `rustc --version` | |
| Java | `java --version 2>&1 \| head -1` | |
| Ruby | `ruby --version` | |
| Swift | `swift --version 2>&1 \| head -1` | |

- "기본 탐지" 항목만 자동으로 탐지한다. 그 외 항목은 사용자가 요청할 때 추가한다.
- `clang` 은 Xcode Command Line Tools 에 포함되어 있다. CLT 미설치 시 최초 호출 시점에 설치 프롬프트가 뜬다.

## 기본 설정 (글로벌 CLAUDE.md에 포함되는 내용)

```
# System Info

- **Machine**: {{머신모델}}
- **CPU**: {{CPU}}
- **RAM**: {{RAM}}GB
- **Storage**: {{스토리지}}
- **GPU**: {{GPU}}
- **Display**: {{디스플레이}}
- **OS**: macOS {{macOS버전}} ({{빌드}}), Kernel {{커널버전}}, {{아키텍처}}

## Development Tools

- **IDE**: {{탐지된 IDE 목록}}
- **Runtime**: {{탐지된 런타임 목록}}
```

## 주의사항
- 기본 셸은 zsh 이다 (macOS 10.15 Catalina 이후). 사용자의 로그인 셸은 `dscl . -read ~/ UserShell` 또는 `echo $SHELL` 로 확인한다.
- 줄바꿈은 LF 가 기본이므로 별도 정규화가 불필요하다.
- Apple Silicon (arm64) 과 Intel (x86_64) 환경에서 Homebrew prefix 가 다르므로 경로를 하드코딩하지 말 것. `brew --prefix` 로 질의할 것.

# 환경 탐지 대상

저장소 `CLAUDE.md` 의 "환경 탐지 세부 규칙" 에 따라 `environment.md` 를 생성·갱신할 때 본 플랫폼에서 탐지하는 툴 목록이다. 상단 "런타임/컴파일러 탐지" 표가 글로벌 CLAUDE.md 템플릿용이라면, 본 섹션은 `environment.md` 용 탐지 대상이다.

## 탐지 대상

| 툴 | 탐지 명령 | 비고 |
| --- | --- | --- |
| Python | `command -v python3 && python3 --version` | Apple 제공 `python3`(Xcode CLT) 과 Homebrew `python3` 이 공존할 수 있음. `command -v` 결과 경로로 구분 기록 |
| Node.js | `command -v node && node --version` | |
| Zsh | `command -v zsh && zsh --version` | 로그인 셸은 `echo $SHELL` 로 추가 기록 |
| Bash | `command -v bash && bash --version \| head -1` | macOS 기본 `/bin/bash` 는 3.2. Homebrew 설치본(`/opt/homebrew/bin/bash` 또는 `/usr/local/bin/bash`) 이 있으면 경로로 구분 기록 |
| PowerShell | `command -v pwsh && pwsh -NoProfile -Command "$PSVersionTable.PSVersion.ToString()"` | Homebrew cask 또는 공식 pkg 로 설치. 미설치 시 "사용 불가" |
| .NET SDK | `command -v dotnet && dotnet --version` | 미설치 시 "사용 불가" 로 기록 |
| Homebrew | `command -v brew && brew --version \| head -1` | Prefix 는 `brew --prefix` 로 함께 기록 |

# 툴 설치 방법

사용자가 특정 툴의 설치를 요청하면 아래 지침에 따라 Homebrew 로 설치한다.

## 공통 사항

- **Homebrew 미설치 시**: 먼저 Homebrew 설치를 안내한다. 공식 설치 명령은 https://brew.sh 참조. 설치 후 쉘 초기화(`eval "$(/opt/homebrew/bin/brew shellenv)"` 또는 `.zprofile` 반영) 가 필요함을 함께 안내한다.
- **PATH 갱신**: Homebrew 는 설치된 바이너리를 `$(brew --prefix)/bin` 에 배치한다. 이미 열려 있는 셸 세션에서 `hash -r` 또는 새 셸을 열어 반영한다.
- **IDE 의 PATH 상속**: VSCode 등 GUI 앱은 macOS 에서 로그인 셸의 환경변수를 상속하지 않을 수 있다. 새 바이너리가 IDE 터미널에서 보이지 않으면 IDE 를 완전히 재시작(Cmd+Q) 하도록 안내한다.
- **cask vs formula**: GUI 앱은 `brew install --cask <이름>`, CLI 툴은 `brew install <이름>` 을 사용한다.
- 설치 완료 후 "환경 탐지 갱신 절차" 를 수행하여 `environment.md` 를 최신화하도록 사용자에게 제안한다.

## Python

- **기본 설치**: `brew install python@3.13`
  - 파이썬은 공식 LTS 개념이 없으며 3.13 은 안정 버전으로 권장한다.
  - Homebrew 는 버전 관리된 formula(`python@3.12`, `python@3.13` 등) 를 제공한다. `python3` 심볼릭 링크는 가장 최근에 설치된 버전을 가리킨다.
- **특정 버전 요구**: `brew install python@3.<minor>` (예: `python@3.12`, `python@3.14`)
- **호출명**: `python3` 이 표준. `python` 은 기본적으로 제공되지 않으므로, `python` 호출이 필요하면 가상환경(`venv`) 을 사용하거나 `pyenv` 도입을 제안한다.
- **Apple 제공 `python3` 과의 공존**: Xcode CLT 가 제공하는 `/usr/bin/python3` 이 있지만 개발용으로는 Homebrew 버전을 우선 사용한다. PATH 우선순위가 `/opt/homebrew/bin` 이 앞이어야 한다.

## Node.js

- **기본 설치 (LTS)**: `brew install node@lts` (또는 특정 메이저 버전 `brew install node@20`)
- **최신 Current 버전**: `brew install node`
  - Current 는 최신 기능이 먼저 들어오지만 수명이 짧다. 특별한 이유가 없으면 LTS 를 권장한다.
- **버전 전환 필요 시**: `nvm` 또는 `fnm` 도입을 제안한다(본 저장소에서는 기본 채택하지 않음).
- **Corepack**: Node 설치에 `corepack` 이 포함되지만 기본 비활성이다. `yarn`/`pnpm` 사용 예정이면 `corepack enable` 을 안내한다.

## PowerShell

- **설치**: `brew install --cask powershell`
- 호출명은 `pwsh`. Windows PowerShell (`powershell`) 은 macOS 에 존재하지 않는다.

## .NET SDK

- **기본 설치 (LTS)**: `brew install --cask dotnet-sdk`
- 특정 버전이 필요하면 공식 installer (.pkg) 사용을 안내한다.

## Mono

Unity 프로젝트를 VSCode 등에서 디버깅할 때 필요하다. Unity 가 번들하는 Mono 런타임과 별개로, IDE 의 디버거가 호스트 시스템의 Mono MDK 를 요구하는 경우가 있다.

- **기본 설치**: `brew install --cask mono-mdk`
  - cask 이름은 `mono-mdk` (MDK = Mono Development Kit). Unity/VSCode 디버깅 용도는 이 패키지가 표준이다.
  - macOS 10.15 이상 필요.
- **`mono` formula 와의 충돌**: `mono-mdk` 설치 시 `/usr/local/bin` (또는 `$(brew --prefix)/bin`) 의 `mono` formula 바이너리가 제거되고 `/private/etc/paths.d/mono-commands` 가 추가된다. 기존에 `brew install mono` 로 formula 버전을 사용 중이었다면 설치 전 사용자에게 고지한다.
- **Visual Studio for Mac 용 별도 cask**: `mono-mdk-for-visual-studio` 는 VS for Mac 전용 변형이며, Unity/VSCode 목적에는 `mono-mdk` 를 사용한다.
