# Agent Teams Skill 구현 계획 (검증 후 진행)

## 전제 조건
- agent-teams-guide-draft.md의 내용이 2차 이상 시도를 통해 안정화된 후 구현
- 가이드 내용이 더 이상 큰 변경 없이 반복 사용 가능한 수준이 되었을 때 착수

## Skill 구조

### 위치
`~/.claude/skills/agent-teams/SKILL.md` (글로벌, 모든 프로젝트에서 사용 가능)

### 프론트매터
```yaml
---
name: agent-teams
description: "사용자가 여러 파일/모듈에 걸친 대규모 작업을 요청하거나, 에이전트 팀으로 병렬 작업을 하려는 경우 호출한다."
disable-model-invocation: true  # 수동 호출 전용 (/agent-teams)
---
```

### SKILL.md 내용 방향
- 판단 기준과 체크리스트만 간결하게 유지 (컨텍스트 절약)
- 상세 가이드는 별도 파일에 두고, SKILL.md 안에서 "상세 내용은 해당 경로를 Read할 것"으로 참조
- 참조 경로: `~/.claude/docs/agent-teams-guide.md` (draft 접미사 제거 후 확정판)

### 의도하는 흐름
1. 사용자가 `/agent-teams` 호출
2. 가이드 기준으로 에이전트 팀 적합 여부 판단
3. 적합하면 계획 수립 → CLAUDE.md 생성 → 사용자 검토 요청
4. 승인 후 단계별 실행 (중간 점검 포함)

### 자동 호출 관련
- 초기에는 `disable-model-invocation: true`로 수동 호출 전용
- 충분히 안정화되면 자동 호출 허용 여부를 재검토
