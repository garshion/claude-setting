# 환경 탐지 대상

저장소 `CLAUDE.md` 의 "환경 탐지 세부 규칙" 에 따라 `environment.md` 를 생성·갱신할 때 본 플랫폼에서 탐지하는 툴 목록이다.

## 탐지 대상

| 툴 | 네이티브 탐지 | WSL 탐지 | 비고 |
| --- | --- | --- | --- |
| Python | `where python` / `where python3` 후 해당 명령으로 `<cmd> --version` | `wsl -- command -v python3 && wsl -- python3 --version` | 호출명(`python` vs `python3`)을 함께 기록 |
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

# 주의사항
- **줄바꿈 정규화**: Write 도구가 LF로 파일을 쓰지만, `.gitattributes`에서 CRLF 강제하는 프로젝트가 있음. 파일 작업 완료 후 반드시 `git add --renormalize .` 실행할 것. 단, `git add`로 먼저 스테이징하면 renormalize가 효과 없으므로, 스테이징 없이 renormalize만 실행할 것.
