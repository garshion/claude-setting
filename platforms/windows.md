# 주의사항
- **줄바꿈 정규화**: Write 도구가 LF로 파일을 쓰지만, `.gitattributes`에서 CRLF 강제하는 프로젝트가 있음. 파일 작업 완료 후 반드시 `git add --renormalize .` 실행할 것. 단, `git add`로 먼저 스테이징하면 renormalize가 효과 없으므로, 스테이징 없이 renormalize만 실행할 것.
