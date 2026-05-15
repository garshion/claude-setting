# C# 코드 규칙

## 공통 스타일 (C++/C#)
- **if 다중 조건**: 줄바꿈 후 `&&`/`||`를 다음 줄 앞쪽에 배치할 것.
- **유효성 검사**: `return 조건1 && 조건2;` 형태의 return 체이닝을 사용하지 말 것. `if (조건) return false; return true;` 형태로 명시적 분기를 사용할 것.
- **enum 네이밍**: `E` 접두사 사용 (예: `EAuthorityLevel`, `EAuthorityCategory`).

## C# 스타일
- 기본 C# 권장 스타일을 따를 것.
- **Top-level statements 사용 금지**: `Main` 메서드를 명시적으로 작성할 것.
- **암시적 using 사용 금지**: `ImplicitUsings`를 비활성화하고, 모든 `using`을 명시적으로 선언할 것.
- **경고 레벨**: 프로젝트 생성 시 기본값으로 두되, 생성 후 `.csproj`의 `WarningLevel`, `AnalysisLevel` 등 현재 설정값을 확인하여 사용자에게 보고하고, 변경 여부를 물을 것.
- **주석 스타일**: summary 주석에서 줄바꿈이 필요하면 `<br/>`을 사용할 것 (IDE 툴팁 줄바꿈 표시용). 설명 주석은 꼼꼼하게 작성할 것.
- **nullable**: nullable 참조 타입 활성화 프로젝트에서 non-nullable 파라미터에 대한 null 체크는 불필요. 추가하지 말 것.

## C# 테스트 규칙

### 테스트 프로젝트 셋업
- **프레임워크**: xUnit. 기존 프로젝트에 다른 프레임워크(NUnit, MSTest)가 있으면 그것을 따를 것.
- **프로젝트명**: `<대상 프로젝트명>.Tests` (기본값).
- **테스트 프로젝트가 없는 경우**: `dotnet new xunit -n <프로젝트명>.Tests` 로 생성 후 솔루션에 추가(`dotnet sln add`). 대상 프로젝트를 참조로 추가(`dotnet add reference`).

### 네이밍
- **테스트 클래스**: `<대상 클래스명>Tests` (예: `OrderServiceTests`)
- **테스트 메서드**: `<메서드명>_<상황>_<기대결과>` (예: `PlaceOrder_EmptyItems_ReturnsFalse`)

### 파일 배치
- 소스와 1:1 대응 (`UserService.cs` -> `UserServiceTests.cs`).
- 테스트 프로젝트 내 디렉토리 구조는 대상 프로젝트의 구조를 따를 것.
