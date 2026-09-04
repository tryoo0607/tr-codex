# tr-codex

개인 kit — **22개 스킬**. `tr-kit-source`에서 `build.sh codex`로 생성(수동 유지 X).

## 스킬

| 스킬 | 발동 |
|---|---|
| `analysis` | 코드베이스 구조·모듈 배선·데이터 흐름을 분석해 온보딩용 HTML 문서로 만들 때 사용한다. "프로젝트 구조 분석", "온보딩 문서". |
| `api-handoff` | 백엔드 API 변경을 프론트엔드 전달용 요청서로 정리할 때 사용한다. "FE 요청서", "API 변경 핸드오프". 프론트엔드 직접 구현에는 사용하지 않는다. |
| `architecture` | 구현 전에 방법론·아키텍처 스타일·디자인 패턴 후보와 trade-off를 검토할 때 사용한다. "어떤 패턴?", "아키텍처 선택". 확정 설계나 코드 작성에는 사용하지 않는다. |
| `career` | 작업·성과를 커리어 기록으로 축적하거나 이력서·면접 재료로 정리할 때 사용한다. "업무 로그", "성과 정리", "이번 달 뭐 했지". |
| `ci` | GitHub Actions workflow를 작성·수정하거나 CI/CD pipeline을 구성할 때 사용한다. "CI 짜줘", "Actions 설정". 다른 CI 제품에는 사용하지 않는다. |
| `convention` | 이름, 코딩 스타일, README·docstring 등 작성 관례를 정하거나 적용할 때 사용한다. "이름 추천", "우리 규칙", "README 형식". |
| `deps` | 의존성 추가·업그레이드·제거와 취약점 audit에 사용한다. "라이브러리 업글", "의존성 최신화", "취약점 점검". |
| `diagram` | 구조·흐름·관계를 Mermaid 다이어그램으로 만들거나 유지할 때 사용한다. "구성도", "시퀀스", "ERD", "상태도". |
| `dive` | 현재 작업과 관련된 주제를 더 깊게 탐구하거나 복습할 때 사용한다. "이거 파보자", "그게 뭔데?", "복습하자". |
| `git` | commit, branch, merge, rebase, PR 등 Git workflow를 다룰 때 사용한다. "커밋해줘", "브랜치 따줘", "PR 만들어줘". worktree 공간 관리는 project가 담당한다. |
| `kit` | tr-kit 자체를 설치·점검·업데이트하거나 skill·component를 저작할 때 사용한다. "kit 업데이트", "스킬 만들어줘", "kit 점검". |
| `knowledge` | 재사용 지식을 Markdown LLM Wiki에 축적·조회·정리할 때 사용한다. "지식으로 남겨", "전에 기록한 것 찾아줘", "지식베이스 점검해줘". |
| `logs` | 애플리케이션·서버·IDE·컨테이너 로그를 찾아 읽고 분석할 때 사용한다. "로그 봐줘", "이 오류 로그 분석". |
| `perf` | 성능 문제를 측정하고 병목을 찾아 최적화할 때 사용한다. "느려", "성능 점검", "최적화". |
| `profile-setup` | tr-kit의 non-secret 로컬 repository·경로·기능 설정을 연결·점검할 때 사용한다. "profile.d 설정", "career 저장소 연결", "profile doctor". |
| `project` | '`~/projects` 아래 project 공간과 local-docs·worktree·backup을 관리하고 프로젝트 지식을 조회·정리할 때 사용한다. "프로젝트 생성", "worktree", "local-docs", "전에 한 작업", "프로젝트 구조", "백업 연결". 코드 내용은 다루지 않는다.' |
| `proofread` | 산문·README·설계 문서를 교정·검수할 때 사용한다. "오타 봐줘", "글 검수". 코드 검토에는 사용하지 않는다. |
| `prototype` | 웹·모바일 UI 아이디어를 단일 HTML prototype·mockup으로 만들 때 사용한다. "화면 시안", "프로토타입", "대시보드 목업". production 구현에는 사용하지 않는다. |
| `review` | 이미 작성된 코드·diff·PR을 품질과 결함 관점에서 검토할 때 사용한다. "코드 리뷰", "PR 검토". 구현 전 구조 파악에는 사용하지 않는다. |
| `secrets` | secret·credential·token 관리와 코드·commit 유출 검사에 사용한다. "secret 검사", "키 관리", "git-crypt 설정". |
| `session` | 세션·작업의 rollover, handoff, resume 또는 무응답 세션 진단에 사용한다. "세션 넘겨줘", "인계", "세션이 멈췄어". |
| `test` | unit·integration·E2E 테스트를 작성·실행하거나 검증 범위를 점검할 때 사용한다. "테스트 써줘", "E2E", "실데이터로 검증". |
