---
name: review-csharp
description: C# 변경분, 지정 커밋, 또는 전체 코드베이스를 다축으로 코드 리뷰하여 마크다운 리포트로 저장합니다
argument-hint: "[<commit> | <from>..<to> | 전체 | all]"
disable-model-invocation: true
model: claude-opus-4-6
allowed-tools: Bash(git *) Bash(date *) Bash(mkdir *) Bash(rm *) Read Write Agent
---

# /review-csharp

C# 변경분 또는 전체 코드베이스에 대해 5개 영역을 서브에이전트로 병렬 분석하여 마크다운 리포트로 저장합니다.

영역: 안정성·동시성·성능 / 보안 / 컨벤션·스타일 / 테스트 커버리지(diff 모드만) / 명세 적합성(조건부, diff 모드만).

## 인자

`$ARGUMENTS`:

- 비어 있음 → 작업 트리(staged + unstaged)를 HEAD와 비교
- `<commit-sha>` → 단일 커밋
- `<from>..<to>` → 커밋 범위
- `전체` 또는 `all` → 전체 코드베이스 분석 (전체 모드)

## 실행 절차

### 0단계: 모드 결정

`$ARGUMENTS`를 소문자로 정규화한 뒤, `전체`, `all` 중 하나이면 → **전체 모드**. 그 외 → **diff 모드**.

### 1단계: 대상 결정

**diff 모드:**

| 인자 | diff 명령 | 변경 파일 명령 | 표시용 대상명 |
| --- | --- | --- | --- |
| 비어 있음 | `git diff HEAD` | `git diff --name-only HEAD` | `working tree vs HEAD` |
| `<sha>` | `git show <sha>` | `git show --name-only --format= <sha>` | `commit <sha>` |
| `<from>..<to>` | `git diff <from>..<to>` | `git diff --name-only <from>..<to>` | `<from>..<to>` |

git 저장소가 아니거나 인자가 잘못된 경우 사용자에게 안내 후 종료.

**전체 모드:**

`git ls-files`로 추적 파일 전체 목록을 획득. 표시용 대상명: `전체 코드베이스`.

### 2단계: 파일 필터링

1단계 결과에서 검사 대상 파일을 필터링.

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

**전체 모드 파일 수 상한**: 검사 대상이 100개를 초과하면 사용자에게 "검사 대상이 N개입니다. 계속하시겠습니까?"라고 확인. 거부 시 종료.

### 3단계: 임시 디렉토리 및 분석 대상 저장

임시 디렉토리: `<프로젝트 루트>/.review-csharp-tmp/` (없으면 `mkdir -p`로 생성)

**diff 모드:**

1단계 diff 명령으로 diff 본문을 획득하여 `.review-csharp-tmp/diff-<타임스탬프>.diff`에 저장.

**전체 모드:**

필터링된 파일 목록을 `.review-csharp-tmp/filelist-<타임스탬프>.txt`에 저장 (줄바꿈 구분).

### 4단계: 명세 처리 (diff 모드 전용)

전체 모드에서는 이 단계를 건너뛴다.

1. 프로젝트 루트의 `step2-plan.md` 존재 여부 확인
2. 존재 시: 사용자에게 다음과 같이 묻고 응답을 기다린다 — "프로젝트 루트의 `step2-plan.md`를 명세로 사용해 검증할까요?"
   - 동의 → 명세 경로 = `step2-plan.md`로 설정, 5단계로
   - 거부 → 3번으로
3. 미존재 또는 거부 시: 다음과 같이 묻고 응답을 기다린다 — "검증할 명세 파일이 있다면 경로를 알려주십시오. 없다면 '없음'이라고 답해주시면 명세 검증은 건너뜁니다."
   - 경로 응답 → 파일 존재를 확인한 뒤 명세 경로로 설정
   - "없음" 또는 동등 표현 → 명세 적합성 에이전트 비활성

### 5단계: 서브에이전트 병렬 디스패치

다음 에이전트를 **한 메시지에 다중 Agent 호출**로 동시 실행한다.

**diff 모드 에이전트 구성** (최대 5개, 명세 비활성 시 4개):

| 영역 | description | subagent_type | model | 프롬프트 파일 |
| --- | --- | --- | --- | --- |
| 안정성·동시성·성능 | C# 안정성·동시성·성능 리뷰 | general-purpose | (부모 상속) | `agents/stability-concurrency-perf.md` |
| 보안 | C# 보안 리뷰 | general-purpose | (부모 상속) | `agents/security.md` |
| 컨벤션·스타일 | C# 컨벤션 리뷰 | general-purpose | sonnet | `agents/convention.md` |
| 테스트 커버리지 | C# 테스트 커버리지 | general-purpose | sonnet | `agents/test.md` |
| 명세 적합성 (조건부) | C# 명세 적합성 | general-purpose | (부모 상속) | `agents/spec.md` |

**전체 모드 에이전트 구성** (3개 고정):

| 영역 | description | subagent_type | model | 프롬프트 파일 |
| --- | --- | --- | --- | --- |
| 안정성·동시성·성능 | C# 안정성·동시성·성능 전체 리뷰 | general-purpose | (부모 상속) | `agents/stability-concurrency-perf.md` |
| 보안 | C# 보안 전체 리뷰 | general-purpose | (부모 상속) | `agents/security.md` |
| 컨벤션·스타일 | C# 컨벤션 전체 리뷰 | general-purpose | sonnet | `agents/convention.md` |

**프롬프트 본문**: 각 에이전트의 `prompt`는 다음 구성으로 만든다.

1. `$CLAUDE_SKILL_DIR/agents/<영역>.md` 파일 내용을 본문에 그대로 포함
2. 그 아래에 다음 형식의 컨텍스트 블록을 붙인다:

```
---
## 컨텍스트

- **모드**: diff | full
- **대상**: <표시용 대상명>
- **임시 디렉토리**: <.review-csharp-tmp 의 절대 경로>
- **변경 파일 목록** (diff 모드만):
  - <파일1>
  - <파일2>
  - ...
- **명세 파일 경로**: <경로 또는 "없음"> (명세 에이전트 전용)

## 분석 대상

diff 모드:
  diff 파일 경로: <절대 경로>
  위 파일을 Read 도구로 읽어 변경 hunks를 파악하라.

전체 모드:
  파일 목록 경로: <절대 경로>
  위 파일을 Read 도구로 읽어 대상 파일 목록을 확인한 뒤, 각 파일을 Read 도구로 읽어 분석하라.

## 분석 범위 엄수 (diff 모드만)

메인 발견 사항의 대상은 **diff의 +/- 라인 (변경된 hunks) 만**이다. 변경 파일 전체나 호출자·피호출자 코드를 추가로 읽는 것은 컨텍스트 파악 목적으로 허용되지만, **변경되지 않은 줄에서 발견한 이슈를 메인 발견 사항에 포함하지 말 것**. 그러한 발견은 변경 흐름과 명백한 연관이 있고 중대한 경우에 한해 Out-of-scope 발견 사항으로 분리 보고한다.
```

컨텍스트 블록 작성 시:

- "분석 대상" 섹션에는 현재 모드에 해당하는 항목만 기재한다 (diff 모드면 diff 파일 경로만, 전체 모드면 파일 목록 경로만).
- 명세 적합성 에이전트가 아닌 경우 "명세 파일 경로" 줄은 생략한다.
- "변경 파일 목록"은 diff 모드에서만 기재한다.
- "분석 범위 엄수" 섹션은 diff 모드에서만 포함한다. 전체 모드에서는 에이전트 프롬프트의 "전체 분석 모드" 지시에 따라 전체 파일을 분석한다.

### 6단계: 결과 취합

각 에이전트는 발견 사항을 `.review-csharp-tmp/findings-<영역명>.md` 파일에 저장하고, 응답으로 파일 경로와 건수 요약을 반환한다.

각 findings 파일을 Read하여 다음 기준으로 통합:

- **정규화**: 영역 / 심각도(Critical/Warning/Suggestion) / 위치(파일:라인) / 제목 / 문제 / 권장 수정 / 변경 외 영역(Out-of-scope) 여부
- **중복 제거**: 같은 위치 + 같은 본질의 발견을 하나로 합치고 영역 라벨을 병합 (예: `[안정성·동시성/보안]`)
- **교차 이슈 보강**: 한 발견이 여러 영역에 해당하면 영역 라벨을 모두 명시
- **분리**: 메인 발견 사항 vs 참고 발견 사항(변경 범위 밖) — 전체 모드에서는 메인만
- **정렬**: 메인 발견은 Critical → Warning → Suggestion 순, 같은 심각도 안에서는 영역별 그룹화

### 7단계: 리포트 작성

1. 현재 시각을 `date +%Y%m%d-%H%M%S`로 획득
2. `$CLAUDE_SKILL_DIR/report-template.md`를 읽어 형식 파악
3. 형식에 맞춰 마크다운 구성. 빈 섹션은 "(없음)", "(명세 미제공)", "(전체 분석 모드에서는 해당 없음)" 등으로 명시
4. 파일명: `code-review-<타임스탬프>.md`
5. 저장 위치: 명령 실행 시점의 **프로젝트 루트**
6. `<프로젝트 루트>/.review-csharp-tmp/` 디렉토리를 `rm -rf`로 제거한다
7. 저장 후 사용자에게 다음 요약을 보고:
   - 저장 경로
   - 심각도별 개수
   - 변경 외 영역 발견 수 (전체 모드에서는 생략)
   - 명세 검증 수행 여부 (전체 모드에서는 "미지원")
