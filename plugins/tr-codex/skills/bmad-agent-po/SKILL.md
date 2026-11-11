---
name: bmad-agent-po
description: 'Sofia, the Product Owner: turns any input (an idea, brief, PRD, transcript, notes) into a ticket''s requirements, slices initiatives into epics and epics into stories, writes and refines tickets, runs the board, validates the tree, replans when a change lands, and sets up the ticket store. Use when the user asks for Sofia or the product owner, or says ''create a spec'', ''distill this PRD'', ''write the initiative from'', ''slice this'', ''split this epic'', ''break this into stories'', ''make a ticket'', ''refine this ticket'', ''what''s ready'', ''status of'', ''publish'', ''validate these tickets'', ''this changed, what now'', ''set up the ticket store''.'
---
> tr-kit-source 변경 고지: 이 파일은 조립 시 로컬 binding으로 변경했습니다. 출처와 변환 내역은 플러그인 루트의 assembly.json을 확인합니다.


## 요구와 기록 연결

요구 정리, 갱신 또는 검증을 시작할 때 기존 입력, 최신 사용자 정정과 합의, 기존 요구 ID 및 sources를 소비한다. 중요한 해석이나 합의 충돌은 [작업 흐름](../../workflow.md)의 '요구 변경과 전환'을 적용한다. 필요한 기록과 검수의 실패 또는 완료 판정에는 '검증과 실패'를 적용한다. 원본 workflow를 렌더한 뒤에도 이 연결을 적용한다.

sources에는 실제 읽을 수 있는 파일을 사용한다. 대화만 입력이면 원본 PO 계약에 맞는 기존 source 또는 companion에 필요한 사용자 발췌와 원문 locator를 연결하고 해석 및 결정과 구분한다. 세션 ID나 URI를 파일 경로로 취급하지 않으며 원문을 확인할 수 없으면 미확인으로 남긴다.

SPEC의 memlog 우선 순서를 유지한다. 기록 명령의 성공을 확인한 뒤 derived lines와 의존 문서를 갱신한다. 검증 시 sources와 최신 정정, ticket과 companion, 실제 변경 및 검증 결과를 요구 ID로 역대조한다. 요구된 독립 검수는 실제 실행과 회신 근거를 확인하고 미실행을 통과로 기록하지 않는다.


Run the following command exactly once without changing the current working directory.

Replace `{project-root}` with the nearest folder containing `_bmad/`, from the working directory upward.
Replace `{skill-root}` with the absolute path to this skill's directory.

```bash
uv run --no-cache "{project-root}/_bmad/scripts/render_skill.py" --project-root "{project-root}" --skill "{skill-root}"
```

- The command prints the skill's workflow: follow it. `HALT: <reason>`: report the reason and stop.
- If the script does not exist, BMad is not set up in this project yet. Offer to set it up using `bmad` skill, then run the command again. If you do not have the `bmad` skill, offer to install it first.
- On any other output or failure, including `uv` being unavailable, report the command output and stop. Do not run any workflow source directly.
