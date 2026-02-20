# claude-setting

Claude Code 글로벌 설정(`~/.claude/CLAUDE.md`)을 관리하는 저장소입니다.

장비를 교체하거나 새로운 환경을 세팅할 때, 이 저장소를 clone한 뒤 Claude Code로 환경에 맞는 글로벌 설정을 자동 생성할 수 있습니다.

## 사용법

1. 이 저장소를 clone합니다.
2. 저장소 디렉토리에서 Claude Code를 실행합니다.
3. 원하는 환경을 지정하여 요청합니다.

### 예시
- `윈도우, VS2026 엔터프라이즈로 설정해줘`
- `Windows, VS2022 Community로 설정해줘`
- `Windows 환경으로 설정해줘` (도구를 자동 탐지)

## 구조

```
common.md              # OS 무관 공통 설정
platforms/
  windows.md           # Windows 전용 설정
tools/
  vs2022.md            # Visual Studio 2022 설정
  vs2026.md            # Visual Studio 2026 설정
CLAUDE.md              # 프로젝트 CLAUDE.md (설치 가이드)
```
