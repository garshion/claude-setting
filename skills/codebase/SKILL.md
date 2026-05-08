---
name: codebase
description: 현재 프로젝트의 코드베이스를 분석하여 .claude/codebase.md를 생성하거나 갱신합니다
argument-hint: "[update]"
disable-model-invocation: true
allowed-tools: Read Write Bash(find *) Bash(git *) Bash(ls *) Bash(mkdir *)
---

# /codebase

프로젝트 코드베이스를 분석하여 `.claude/codebase.md` 를 생성하거나 갱신합니다.
**이 파일은 Claude Code 전용입니다. 인간이 읽는 문서가 아닙니다.**

## 실행 모드

- **인자 없음 (`/codebase`)**: 전체 분석 → `.claude/codebase.md` 전체 재작성
- **`update` (`/codebase update`)**: 변경된 파일 중심으로 관련 섹션만 갱신

---

## 전체 분석 절차

### 1단계: 프로젝트 구조 파악

다음 명령으로 전체 구조를 확인한다:

```
find . -maxdepth 5 \
  -not -path './.git/*' \
  -not -path './bin/*' \
  -not -path './obj/*' \
  -not -path './node_modules/*' \
  -not -path './packages/*' \
  -not -path './.nuget/*'
```

### 2단계: 프로젝트 타입 감지

| 타입 | 판별 기준 |
|------|-----------|
| C# | `.sln` 또는 `.csproj` 파일 존재 |
| C++ | `CMakeLists.txt` 또는 `.vcxproj` 파일 존재 |
| 기타 | 주요 파일 확장자로 판단 |

### 3단계: 핵심 파일 읽기

다음 파일을 읽어 구조를 파악한다:
- 솔루션·프로젝트 파일 (`.sln`, `.csproj`, `CMakeLists.txt`)
- 진입점 파일 (`Program.cs`, `Startup.cs`, `main.cpp` 등)
- 인터페이스·엔티티 정의 파일 (Core/Domain 레이어)
- DI 등록 파일

### 4단계: .claude/codebase.md 작성

`$CLAUDE_SKILL_DIR/format.md` 를 읽고 그 형식에 따라 `.claude/codebase.md` 를 작성한다.
`.claude/` 디렉토리가 없으면 먼저 생성한다.

### 5단계: gitignore 확인

프로젝트에 `.gitignore` 가 있고 `.claude/codebase.md` 항목이 없으면 추가 여부를 사용자에게 안내한다.
(자동으로 추가하지 않고 안내만)

---

## 부분 갱신 절차 (`update`)

### 1단계: 기존 파일 확인

`.claude/codebase.md` 가 없으면 전체 분석으로 전환한다.

### 2단계: 변경 파일 확인

```
git diff --name-only HEAD~1 HEAD
```

git 저장소가 아니거나 변경 파일이 없으면 "변경 없음" 을 보고하고 종료한다.

### 3단계: 변경된 파일 읽기

변경된 소스 파일을 읽어 어떤 섹션이 영향받는지 판단한다.

### 4단계: 관련 섹션만 수정

영향받는 섹션(모듈 맵·핵심 타입·비명시적 컨벤션)만 갱신한다.
**섹션 전체를 재작성하지 말고, 변경된 항목만 추가·수정·삭제한다.**
파일 상단의 `**갱신:** YYYY-MM-DD` 날짜를 오늘 날짜로 업데이트한다.
