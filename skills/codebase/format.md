# codebase.md 포맷 정의

이 문서는 `/codebase` 실행 시 생성하는 `.claude/codebase.md` 의 포맷을 정의합니다.
언어·프레임워크·프로젝트 구조에 따라 항목을 적절히 조정할 것.
아래 C# 예시는 참고용이며, 실제 코드베이스에 맞게 판단하여 적용.

---

## 포맷 구조

```markdown
# 코드베이스 인덱스
<!-- Claude Code 자동 생성. 직접 편집하지 말 것. /codebase 로 갱신. -->
**갱신:** YYYY-MM-DD

## 탐색 제외
- `<경로>` — <이유>. <절대 탐색 금지 | 읽기만, 수정 금지>

## 모듈 맵
- `<경로>` — <한 줄 책임 설명>

## 핵심 타입
- `<TypeName>` — <파일 경로>

## 비명시적 컨벤션
- <코드만 보고는 파악 불가한 규칙·제약·패턴>
```

### 섹션별 작성 기준

**탐색 제외**
- 빌드 출력, 패키지 캐시, 자동 생성 코드처럼 분석 불필요한 경로 열거
- 각 항목에 이유와 제한 수준(절대 탐색 금지 / 읽기만, 수정 금지)을 명시

**모듈 맵**
- 프로젝트·네임스페이스·디렉토리 단위로 책임을 한 줄로 설명
- 의존 관계가 명확하면 함께 기술 (예: "Controller → Service 패턴")

**핵심 타입**
- 인터페이스, 엔티티, 서비스, 컨텍스트 등 작업 시 자주 참조할 타입
- 이름 → 파일 경로 형식. 전체 목록이 아닌 핵심 5~15개

**비명시적 컨벤션**
- 코드를 읽어도 파악 불가한 것만 기록
- 예: DI 등록 위치, 에러 처리 집중 지점, 금지 패턴, 파일 쌍 관계

---

## C# 예시

```markdown
# 코드베이스 인덱스
<!-- Claude Code 자동 생성. 직접 편집하지 말 것. /codebase 로 갱신. -->
**갱신:** 2026-05-08

## 탐색 제외
- `bin/`, `obj/` — 빌드 출력. 절대 탐색 금지
- `packages/`, `.nuget/` — NuGet 캐시. 절대 탐색 금지
- `*.Designer.cs`, `*.g.cs` — 자동 생성 코드. 읽기만, 수정 금지

## 모듈 맵
- `MyApp.sln` — 솔루션 루트. 4개 프로젝트 포함
- `src/MyApp.Api/` — ASP.NET Core Web API 진입점. Controller → Service 패턴
- `src/MyApp.Core/` — 도메인 모델·인터페이스. 외부 의존성 없음
- `src/MyApp.Infrastructure/` — DB(Dapper), 외부 API 구현체
- `tests/MyApp.Tests/` — xUnit. 통합 테스트는 Integration/ 하위

## 핵심 타입
- `UserEntity` — src/MyApp.Core/Entities/UserEntity.cs
- `IUserRepository` — src/MyApp.Core/Interfaces/IUserRepository.cs
- `UserService` — src/MyApp.Core/Services/UserService.cs

## 비명시적 컨벤션
- DI는 Program.cs 에서만 등록. 다른 곳에서 직접 인스턴스화 금지
- 에러 처리는 GlobalExceptionHandler.cs 에서 집중 처리
- partial class 파일은 쌍으로 존재. 한 쪽 수정 시 다른 쪽도 확인
```
