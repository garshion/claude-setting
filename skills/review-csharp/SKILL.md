---
name: review-csharp
description: C# 변경분 또는 지정 커밋을 다축으로 코드 리뷰하여 마크다운 리포트로 저장합니다
argument-hint: "[<commit> | <from>..<to>]"
disable-model-invocation: true
model: claude-opus-4-7
allowed-tools: Bash(git *) Bash(date *) Read Write Agent
---

# /review-csharp

C# 변경분에 대해 6개 영역을 서브에이전트로 병렬 분석하여 마크다운 리포트로 저장합니다.

영역: 안정성·정확성·예외·리소스 / 동시성·성능 / 보안 / 컨벤션·스타일 / 테스트 커버리지 / 명세 적합성(조건부).

## 인자

`$ARGUMENTS`:

- 비어 있음 → 작업 트리(staged + unstaged)를 HEAD와 비교
- `<commit-sha>` → 단일 커밋
- `<from>..<to>` → 커밋 범위

## 실행 절차

### 1단계: git 명령 결정

| 인자 | diff 명령 | 변경 파일 명령 | 표시용 대상명 |
| --- | --- | --- | --- |
| 비어 있음 | `git diff HEAD` | `git diff --name-only HEAD` | `working tree vs HEAD` |
| `<sha>` | `git show <sha>` | `git show --name-only --format= <sha>` | `commit <sha>` |
| `<from>..<to>` | `git diff <from>..<to>` | `git diff --name-only <from>..<to>` | `<from>..<to>` |

git 저장소가 아니거나 인자가 잘못된 경우 사용자에게 안내 후 종료.

### 2단계: 변경 추출과 파일 필터링

위 명령으로 diff 본문과 변경 파일 전체 목록을 획득.

**검사 대상 포함**:

- 코드: `*.cs`, `*.cshtml`, `*.razor`
- 프로젝트/빌드: `*.csproj`, `*.sln`, `Directory.Build.props`, `Directory.Packages.props`
- 설정: `appsettings*.json`, `web.config`, `app.config`
- 리소스/UI: `*.resx`, `*.xaml`, `*.axaml`

**제외**:

- 순수 프런트엔드: `*.css`, `*.scss`, `*.less`, 정적 `*.html`, 서버사이드 아닌 `*.js`/`*.ts`, 이미지·폰트
- 빌드 산출물: `bin/`, `obj/` 하위
- 자동 생성/잠금: `*.g.cs`, `*.Designer.cs`, `*.lock`

필터링 후 검사 대상이 0개면 "검사 대상 파일이 없습니다"를 안내하고 종료.

### 3단계: 명세 처리 (인터랙티브)

1. 프로젝트 루트의 `step2-plan.md` 존재 여부 확인
2. 존재 시: 사용자에게 다음과 같이 묻고 응답을 기다린다 — "프로젝트 루트의 `step2-plan.md`를 명세로 사용해 검증할까요?"
   - 동의 → 명세 경로 = `step2-plan.md`로 설정, 5단계로
   - 거부 → 3번으로
3. 미존재 또는 거부 시: 다음과 같이 묻고 응답을 기다린다 — "검증할 명세 파일이 있다면 경로를 알려주십시오. 없다면 '없음'이라고 답해주시면 명세 검증은 건너뜁니다."
   - 경로 응답 → 파일 존재를 확인한 뒤 명세 경로로 설정
   - "없음" 또는 동등 표현 → 명세 적합성 에이전트 비활성

### 4단계: 서브에이전트 병렬 디스패치

다음 에이전트를 **한 메시지에 다중 Agent 호출**로 동시 실행한다. 명세 비활성 시 명세 적합성 에이전트는 제외(5개 동시 실행).

| 영역 | description | subagent_type | model | 프롬프트 파일 |
| --- | --- | --- | --- | --- |
| 안정성·정확성·예외·리소스 | C# 안정성 리뷰 | general-purpose | opus | `agents/stability.md` |
| 동시성·성능 | C# 동시성·성능 리뷰 | general-purpose | opus | `agents/concurrency-perf.md` |
| 보안 | C# 보안 리뷰 | general-purpose | opus | `agents/security.md` |
| 컨벤션·스타일 | C# 컨벤션 리뷰 | Explore | sonnet | `agents/convention.md` |
| 테스트 커버리지 | C# 테스트 커버리지 | Explore | sonnet | `agents/test.md` |
| 명세 적합성 (조건부) | C# 명세 적합성 | general-purpose | opus | `agents/spec.md` |

각 에이전트의 `prompt`는 다음 구성으로 만든다:

1. `$CLAUDE_SKILL_DIR/agents/<영역>.md` 파일 내용을 본문에 그대로 포함
2. 그 아래에 다음 형식의 컨텍스트 블록을 붙임:

```
---
## 컨텍스트

- **대상**: <표시용 대상명>
- **변경 파일 목록**:
  - <파일1>
  - <파일2>
  - ...
- **명세 파일 경로**: <경로 또는 "없음"> (명세 에이전트 전용)

## diff 본문

<여기에 diff 전체 삽입>
```

명세 적합성 에이전트가 아닌 경우 "명세 파일 경로" 줄은 생략한다.

### 5단계: 결과 취합

각 에이전트의 응답을 다음 기준으로 통합:

- **정규화**: 영역 / 심각도(Critical/Warning/Suggestion) / 위치(파일:라인) / 제목 / 문제 / 권장 수정 / 변경 외 영역(Out-of-scope) 여부
- **중복 제거**: 같은 위치 + 같은 본질의 발견을 하나로 합치고 영역 라벨을 병합 (예: `[안정성/동시성]`)
- **교차 이슈 보강**: 한 발견이 여러 영역에 해당하면 영역 라벨을 모두 명시
- **분리**: 메인 발견 사항 vs 참고 발견 사항(변경 범위 밖)
- **정렬**: 메인 발견은 Critical → Warning → Suggestion 순, 같은 심각도 안에서는 영역별 그룹화

### 6단계: 리포트 작성

1. 현재 시각을 `date +%Y%m%d-%H%M%S` 로 획득
2. `$CLAUDE_SKILL_DIR/report-template.md` 를 읽어 형식 파악
3. 형식에 맞춰 마크다운 구성. 빈 섹션은 "(없음)" 또는 "(명세 미제공)" 으로 명시
4. 파일명: `code-review-<타임스탬프>.md`
5. 저장 위치: 명령 실행 시점의 **프로젝트 루트**
6. 저장 후 사용자에게 다음 요약을 보고:
   - 저장 경로
   - 심각도별 개수
   - 변경 외 영역 발견 수
   - 명세 검증 수행 여부
