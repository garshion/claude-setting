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
