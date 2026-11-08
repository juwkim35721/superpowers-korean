---
name: diagnosing-superpowers
description: superpowers 세션이 잘못되어 인간 파트너(human partner)가 그 원인을 알고 싶어 할 때 — 반복된 작업, 무시된 계획, 시행착오(stumbles), 부실한 결과, 작동하지 않은 스킬, "시간이 너무 오래 걸렸다", "왜 이렇게 비용이 많이 들었나", "지금 뭘 하고 있는 건가" — 또는 어떤 하네스(harness)에서든 현재 세션이나 ID/경로로 식별된 과거 세션에 대해 superpowers 메인테이너에게 제출할 버그 리포트를 작성하고 싶어 할 때 사용합니다.
---

# Superpowers 진단하기 (Diagnosing Superpowers)

## 개요 (Overview)

인간 파트너(human partner)와 함께 세션에서 무엇이 잘못되었는지 명확히 짚어내고, 디스크에 저장된 트랜스크립트(transcripts)를 읽은 뒤, 증거와 함께 무슨 일이 일어났는지 보고합니다. 당신은 보고(report)만 하며, superpowers 자체를 진단(diagnose)하지는 않습니다. 번들(bundle)이나 이슈를 분류(triage)하는 담당자가 superpowers를 변경할지 여부를 결정합니다.

**핵심 원칙 (Core principle):** 모든 발견 사항(finding)은 `path:line`을 인용해야 합니다. 인용이 없으면 발견 사항도 아닙니다. 모든 수치는 기억이 아니라 트랜스크립트나 당신이 직접 실행한 명령어에서 나와야 합니다.

## 워크플로우 (Workflow)

각 단계마다 todo를 생성하세요. 5~7단계는 명시된 조건이 충족될 때만 실행합니다.

1. **문제 접수 (Problem intake).** 세션(들), 알고 있다면 턴 범위(turn range), 인간 파트너가 기대했던 것, 실제로 일어난 일, 그리고 파트너가 중요하게 여기는 관측 지표(소요 시간(wall-clock), 토큰, 반복된 작업, 특정 단일 행동)를 명시한 문제 진술문(problem statement)을 작성할 수 있을 때까지 한 번에 하나씩 질문하세요. "시간이 너무 오래 걸렸다"는 불만(complaint)이지 문제 진술문이 아닙니다. 목표가 superpowers 버그 리포트인지 여부도 기록해 두세요.
2. **위치 파악 (Locate).** `references/session-discovery.md`를 사용하여 각 세션을 검증된 절대 파일시스템 경로로 확인(resolve)하세요. 과거 세션은 첫 번째 프롬프트와 타임스탬프를 인용하여 맞는지 확인받고, 제외(reject)한 모든 후보 세션을 그 이유와 함께 나열하거나 없으면 "none"이라고 적으세요. 서브에이전트 트랜스크립트들도 나열합니다. `~/.superpowers/diagnosing-superpowers/<session-id>/` 디렉토리를 생성하고 인간 파트너에게 경로를 알려준 뒤, 환경 및 스킬 관측에 대한 출처(provenance) 규칙을 준수하여 그곳에 `templates/case.md`를 작성하세요.
3. **분류 및 분석 (Triage).** 보고된 문제 주변 영역을 먼저 직접 읽으세요. 그런 다음 분석 차원(dimension)마다 하나씩 분석가 서브에이전트(analyst subagent)를 병렬로 디스패치합니다. 각 서브에이전트에게는 케이스 파일 경로, `prompts/analyst-common.md`, 그리고 `prompts/` 아래의 차원별 파일 중 하나(`skill-timeline.md`, `plan-adherence.md`, `repeated-work.md`, `stumbles.md`, `quality-evidence.md`, `request-conflicts.md`, `cost-and-time.md`)를 전달합니다. 트랜스크립트가 긴 경우 차원을 턴 범위별로 분할하세요. 반환된 발견 사항 중 `path:line` 인용이 없는 것은 모두 버립니다.
4. **보고서 작성 (Report).** `templates/report.md`의 모든 섹션을 순서대로 채워 작업 공간(workspace)에 저장하고, 내용을 보여준 뒤 경로를 안내하세요. 인용된 내용이 실제로 무엇을 증명하는지 확인하고 뒷받침하는 케이스를 보존하세요. 심볼릭 링크 별칭(symlink alias)은 불필요한 중복 복사본이 아닙니다.
5. **GitHub 이슈 (GitHub issues)** — 보고서 §7에 가능성 있음(possible) 또는 높음(likely)으로 기재되었거나, 인간 파트너가 요청한 경우에 실행합니다. `references/github-issues.md`에 따라 열려 있는 이슈와 닫힌 이슈 모두에서 해당 증상을 검색하세요. 일치하는 이슈들을 보여주고 가장 가까운 이슈에 보고서를 추가할 것을 제안합니다. 일치하는 것이 없다면 `templates/issue.md`를 채워 작업 공간에 저장하고, 정확한 원문을 보여준 뒤, 승인을 받은 후에만 이슈를 생성하세요. `gh` CLI는 파일을 첨부할 수 없으므로, 번들이 존재한다면 파트너가 브라우저에서 직접 첨부할 수 있도록 파일 경로를 알려주세요.
6. **내보내기 (Export)** — 인간 파트너가 번들(bundle)을 요청했을 때만 실행하며, 요청받지 않았는데 임의로 생성하지 마세요. 문제 접수(intake) 단계의 목표가 버그 리포트였다면, 요청 시 민감 정보가 제거된(scrubbed) 번들을 만들 수 있다고 한 번만 언급하고 기다리세요. 각 수준에 무엇이 포함되는지 설명하며 정보 가림 수준(redaction level)을 질문합니다: `skeleton`(도구 실행 결과 본문 제외), `evidence`(인용된 이벤트의 본문만 포함), `full`(전체 포함). `templates/bundle-README.md`에 따라 번들을 구성하고, `prompts/scrub.md`를 디스패치한 뒤 `prompts/scrub-audit.md`를 디스패치하며, 감사(audit) 결과가 `CLEAN`을 반환할 때까지 두 과정을 반복합니다. 최종 스크럽 로그(scrub log), 파일 목록, 개인정보 보호 및 증거 점검 결과를 보여주기 전에 번들 템플릿의 증거 확인 및 대조(reconciliation) 절차를 완료하세요. 승인을 받은 후에만 아카이브(`zip -r` 또는 `tar -czf`)를 생성합니다. 아카이브 경로와 함께 그 안에 무엇이 들어 있는지 밝히고, 치환된 내역은 스크럽 로그를 참고하도록 안내하며, 스크럽 과정에서도 미처 걸러지지 않은 정보가 있을 수 있으므로 공유하기 전에 반드시 모든 파일을 직접 검토해야 한다고 말하세요.
7. **유사 세션 검색 (Similar sessions)** — 요청받았을 때만 실행합니다. 확인된 발견 사항들을 시그니처(signature)로 변환하고, 수정 시간(mtime)과 크기를 기준으로 후보 세션들을 나열한 뒤, 마커 줄 번호(marker line numbers)를 찾고, 후보마다 병렬로 `prompts/similar-session.md`를 디스패치한 후 보고서 §9를 추가합니다.

## 빠른 참조 (Quick reference)

일곱 개의 분석가 에이전트는 항상 모두 실행됩니다. 아래 표는 3단계에서 당신이 직접 먼저 읽어야 할 영역과 최종 결론(verdict)에서 맨 앞에 내세워야 할 발견 사항을 알려줍니다.

| 불만 사항 (Complaint) | 먼저 읽고 앞세울 항목 (Read first, lead with) |
|---|---|
| "시간이 너무 오래 걸렸다 (It took too long)" | `cost-and-time`, `stumbles` |
| "왜 이런 불필요한 추가 작업을 했나? (Why did it do this extra work?)" | `repeated-work`, `plan-adherence` |
| "왜 이렇게 비용이 많이 들었나? (Why is it so expensive?)" | `cost-and-time` |
| "(아직 실행 중인데) 대체 뭘 하고 있는 건가? (What the hell is it doing?)" | `skill-timeline`; 커버리지에 진행 중(in-progress)임을 명시 |
| "계획을 무시했다 (It ignored the plan)" | `plan-adherence`, 컨텍스트 압축(compaction) 줄부터 먼저 확인 |
| "스킬 X가 전혀 작동하지 않았다 (Skill X never fired)" | `skill-timeline` |

## 절대 규칙 (Hard rules)

- **컨텍스트 안전성 (Context safety).** 트랜스크립트 한 줄이 1메가바이트에 달할 수 있습니다. 모든 세션 파일에 대해 매번 `references/context-safety.md`를 따르세요.
- **읽기 전용 (Read-only).** 세션 파일을 절대 수정, 이동, 삭제하지 마세요.
- **서브에이전트에게는 정확한 경로 전달 (Exact paths to subagents).** 서브에이전트의 "현재 세션"은 서브에이전트 자신의 세션입니다. 반드시 절대 경로와 ID를 전달하세요.
- **인간의 프롬프트만 인정 (Human prompts only).** 훅(hook) 출력, 시스템 리마인더, 도구 실행 결과는 인간 파트너가 한 말이 아닙니다. 서브에이전트 트랜스크립트에서 "user"는 부모 에이전트(parent agent)입니다.
- **superpowers 자체를 진단하지 말 것 (No superpowers diagnosis).** 보고서 §7은 관련 여부(involvement)만 기술하고 멈춥니다. 스킬의 결함을 단정하거나 변경 사항을 제안하지 마세요. 인간 파트너가 수정안을 내놓으라고 재촉하더라도 이 규칙은 면제되지 않습니다. 이슈 등록 단계를 안내하고 요청 시 번들을 생성할 수 있다고 언급하세요. 인간 파트너에게 조언(advice)도 하지 마세요.
- **승인 게이트 (Approval gates).** 인간 파트너가 스크럽 로그와 파일 목록을 확인하기 전에는 아카이브를 생성하지 마세요. 파트너가 정확한 원문을 승인하기 전에는 이슈나 코멘트를 등록하지 마시고, 플레이스홀더 요약이 들어간 상태로도 등록하지 마세요.
- **분석 전 문제 접수 필수 (Intake before analysis).** 인간 파트너가 답변하기 전에는 2~7단계의 어떤 작업도 시작하지 마세요. 파트너가 자리를 비웠다면 질문만 적어두고 멈추세요. 당신이 파트너를 대신해 재구성한 진술문은 답변이 아닙니다. 단, 이미 범위가 명확히 지정된 요청 — 특정 단일 이벤트, 현재 실행 중인 작업 확인, 또는 실행할 분석 자체를 지정한 요청 — 은 그 자체로 문제 진술문이므로 먼저 답한 뒤 질문하세요. 세션 전체에 대한 막연한 "왜(why)"는 불만(complaint)입니다.

## 위험 신호 (Red Flags)

| 생각 (Thought) | 현실 (Reality) |
|---------|---------|
| "문제가 명백하니 접수(intake) 단계는 건너뛰자" | 문제 진술문이 모든 분석의 범위를 결정합니다. 질문하세요. |
| "파트너가 자리를 비웠으니 내가 진술문을 재구성해야지" | 파트너가 무엇을 원했는지는 당신이 재구성할 수 없습니다. 질문을 적어두고 멈추세요. |
| "일단 전체를 다 훑어보고 마지막에 물어봐야지" | 범위가 지정되지 않은 전체 스윕(sweep)은 엉뚱한 질문에 파트너의 예산을 낭비합니다. 먼저 질문하세요. |
| "버그 리포트를 원하니 지금 바로 번들을 만들어야지" | 번들은 패키징된 파트너의 세션 데이터입니다. 파트너가 명시적으로 요청할 때만 만드세요. |
| "구조 변경도 필요 없는 작고 국소적인 수정인데 뭐" | 아무리 작아도 당신이 결정할 일이 아닙니다. 증거만 보고하세요. 판단은 분류 담당자(triager)가 합니다. |
| "토큰당 가격은 널리 알려져 있잖아" | 트랜스크립트에서 직접 계산하지 않은 숫자는 지어낸 것입니다. 출처를 인용하거나 빼세요. |
