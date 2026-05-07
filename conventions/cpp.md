# C++ 코드 규칙

## 공통 스타일 (C++/C#)
- **if 다중 조건**: 줄바꿈 후 `&&`/`||`를 다음 줄 앞쪽에 배치할 것.
- **유효성 검사**: `return 조건1 && 조건2;` 형태의 return 체이닝을 사용하지 말 것. `if (조건) return false; return true;` 형태로 명시적 분기를 사용할 것.
- **enum 네이밍**: `E` 접두사 사용 (예: `EAuthorityLevel`, `EAuthorityCategory`).

## C++ 네이밍
- **클래스**: PascalCase (예: `SceneManager`)
- **변수**: camelCase, 멤버변수는 `m` 접두사 (예: `mCurrentScene`)
- **상수**: ALL_CAPS (예: `MAX_PLAYERS`)
- **파일**: 클래스명과 일치하는 PascalCase
- **함수**: PublicFunc(), ProtectedFunc(), _PrivateFunc() 형식. private 함수는 _ 를 붙인 PascalCase.

## 멤버 선언 순서
- public, protected, private 순서로 변수 먼저 선언, public, protected, private 순서로 함수 선언.
