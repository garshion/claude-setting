# Linux (Debian/Ubuntu 계열) 설정

## 경로 탐지

### OS 정보
- 배포판: `lsb_release -ds` 또는 `/etc/os-release`의 `PRETTY_NAME`
- 커널: `uname -r`

### IDE 탐지
| IDE | 탐지 명령 | 기본 탐지 |
|-----|-----------|-----------|
| VSCode | `which code && code --version \| head -1` | O |
| Cursor | `which cursor && cursor --version \| head -1` | |
| JetBrains IDEs | `which idea`, `which clion`, `which rider`, `which goland` | |

### 런타임/컴파일러 탐지
| 도구 | 탐지 명령 | 기본 탐지 |
|------|-----------|-----------|
| .NET | `dotnet --version` | O |
| GCC/G++ | `gcc --version \| head -1` | O |
| Clang/Clang++ | `clang --version \| head -1` | O |
| Python | `python3 --version` | O |
| Node.js | `node --version` | |
| Go | `go version` | |
| Rust | `rustc --version` | |
| Java | `java --version 2>&1 \| head -1` | |

- "기본 탐지" 항목만 자동으로 탐지한다. 그 외 항목은 사용자가 요청할 때 추가한다.

## 기본 설정 (글로벌 CLAUDE.md에 포함되는 내용)

```
# System Info

- **Machine**: {{머신모델}}
- **CPU**: {{CPU}}
- **RAM**: {{RAM}}GB
- **Storage**: {{스토리지}}
- **GPU**: {{GPU}}
- **Display**: {{디스플레이}}
- **OS**: {{배포판}}, Kernel {{커널버전}}

## Development Tools

- **IDE**: {{탐지된 IDE 목록}}
- **Runtime**: {{탐지된 런타임 목록}}
```

## 주의사항
- 줄바꿈은 LF가 기본이므로 별도 정규화 불필요

# 환경 탐지 대상

저장소 `CLAUDE.md` 의 "환경 탐지 세부 규칙" 에 따라 `environment.md` 를 생성·갱신할 때 본 플랫폼에서 탐지하는 툴 목록이다. 상단 "런타임/컴파일러 탐지" 표가 글로벌 CLAUDE.md 템플릿용이라면, 본 섹션은 `environment.md` 용 탐지 대상이다.

## 탐지 대상

| 툴 | 탐지 명령 | 비고 |
| --- | --- | --- |
| Python | `command -v python3 && python3 --version` | `python2` 는 탐지 대상 아님 |
| Node.js | `command -v node && node --version` | |
| Bash | `command -v bash && bash --version \| head -1` | 기본 셸은 `echo $SHELL` 로 추가 기록 |
| .NET SDK | `command -v dotnet && dotnet --version` | 미설치 시 "사용 불가" 로 기록 |
