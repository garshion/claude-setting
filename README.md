# claude-setting

Claude Code 글로벌 설정(`~/.claude/CLAUDE.md`)을 관리하는 공용 저장소입니다.

장비를 교체하거나 새로운 환경을 세팅할 때, 이 저장소를 clone한 뒤 Claude Code로 환경에 맞는 글로벌 설정을 자동 생성할 수 있습니다.

## 사용법

1. 이 저장소를 clone합니다.
2. 저장소 디렉토리에서 Claude Code를 실행합니다.
3. 원하는 환경을 지정하여 요청합니다.

### 예시
- `윈도우, VS2026 엔터프라이즈로 설정해줘`
- `Windows, VS2022 Community로 설정해줘`
- `Windows 환경으로 설정해줘` (도구를 자동 탐지)
- `Linux 환경으로 설정해줘`

## 구조

```
common.md              # OS 무관 공통 설정 (코드 규칙, 커뮤니케이션 스타일 등)
platforms/
  windows.md           # Windows 전용 설정
  linux.md             # Linux 전용 설정
tools/
  vs2022.md            # Visual Studio 2022 설정
  vs2026.md            # Visual Studio 2026 설정
CLAUDE.md              # 프로젝트 CLAUDE.md (설치 절차 가이드)
```

## 설치 절차 요약

1. OS 확인 → 해당 `platforms/*.md` 선택
2. 개발 도구 탐지 → `tools/*.md` 선택 및 플레이스홀더 치환
3. 글로벌 CLAUDE.md 조합 및 작성 (기존 파일이 있으면 섹션별 비교 후 변경분만 갱신)
4. 결과 보고

상세 절차는 [CLAUDE.md](CLAUDE.md)를 참조하세요.
