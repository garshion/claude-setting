# claude-setting

Claude Code 글로벌 설정(`~/.claude/`)을 관리하는 저장소입니다.

장비를 교체하거나 새로운 환경을 세팅할 때, 이 저장소를 clone한 뒤 Claude Code로 환경에 맞는 글로벌 설정을 자동 생성할 수 있습니다.

## 사용법

1. 이 저장소를 clone합니다.
2. 저장소 디렉토리에서 Claude Code를 실행합니다.
3. 원하는 환경을 지정하여 요청합니다.

### 예시
- `윈도우, VS2026 엔터프라이즈로 설정해줘`
- `Windows, VS2022 Community로 설정해줘`
- `Windows 환경으로 설정해줘` (도구를 자동 탐지)
- `macOS 환경으로 설정해줘`
- `Linux 환경으로 설정해줘`

이미 설치된 환경을 최신 설정으로 갱신하려면:
- `업데이트해줘` 또는 `설정 최신화해줘`

## 구조

```
common.md              # 항상 로드되는 최소 기본 지침
coding-guidelines.md   # 코딩 가이드라인
conventions/
  cpp.md               # C++ 코드 규칙
  csharp.md            # C# 코드 규칙
platforms/
  windows.md           # Windows 전용 설정
  mac.md               # macOS 전용 설정
  linux.md             # Linux 전용 설정
tools/
  vs2022.md            # Visual Studio 2022 설정
  vs2026.md            # Visual Studio 2026 설정
skills/
  codebase/            # /codebase — 프로젝트 코드베이스 인덱스 생성·갱신
  review-csharp/       # /review-csharp — C# 변경분 다축 코드 리뷰 (안정성·동시성·보안·컨벤션·테스트·명세)
hooks/
  websearch-allow.json # WebSearch 자동 허용 훅
permissions/
  webfetch-allow.json  # WebFetch 권한 허용
docs/                  # 설계·참고 문서
CLAUDE.md              # 설치·업데이트 절차 가이드
```

## 설치 절차 요약

1. OS 확인 → `platforms/*.md` 선택
2. 개발 도구 탐지 → `tools/*.md` 선택 및 플레이스홀더 치환
3. 환경 탐지 (Python, Node.js 등) → `environment.md` 생성
4. `~/.claude/imports/claude-setting/` 에 모듈 파일 설치, `claude-setting.md` 집약 파일 생성
5. `~/.claude/CLAUDE.md` 에 `@claude-setting.md` 임포트 라인 보장
6. Skills / Commands 설치
7. Hooks 설치
8. Permissions 설치
9. 환경 변수 적용
10. 결과 보고

상세 절차는 [CLAUDE.md](CLAUDE.md)를 참조하세요.
