Title: <skill or symptom>: <one-line observable> (<harness>)

- [x] I searched existing issues and this is not a duplicate (searched: <query terms>; closest: <#n title, or "none">)

## Who's filing this report? (required)

<이 세션의 모델 및 하네스>, filing on behalf of <인간 파트너(human partner)가 표기되길 원하는 이름, 또는 "my human partner">, who asked: "<진단이나 이슈 등록으로 이어진 이 세션에서의 파트너 요청 원문, 민감정보 제거됨(scrubbed)>"

## Human-authored summary (required, under 200 words)

<무엇이 잘못되었고 어떻게 작동했어야 하는지에 대해 인간 파트너(human partner)가 직접 자신의 말로 작성한 내용을 그대로 붙여넣으세요. 파트너에게 작성을 요청하세요. 여기에 플레이스홀더가 남아 있다는 것은 아직 이슈를 제출할 준비가 되지 않았다는 뜻입니다. 내용이 너무 길다면 직접 문장을 고치지 말고 파트너에게 줄여달라고 요청하세요.>

## Environment (required)

| Field | Value | Provenance / supporting evidence |
|-------|-------|-------------------------------|
| Superpowers version | <version> (<sha or "not a checkout">) | <historical evidence / unverified snapshot / current observation / unknown>; <location> |
| Harness (Claude Code, Cursor, etc.) | <harness> | <label>; <location> |
| Harness version | <version> | <label>; <location> |
| Your model + version | <model ids seen> | <label>; <location> |
| All plugins installed | <list> | <label>; <location> |
| OS + shell | <os version>, <shell> | <label>; <location> |

## Is this a Superpowers issue or a platform issue?

- [ ] I confirmed this issue does not occur without Superpowers installed

How I verified that this issue does not reproduce without Superpowers:
<"Not tried", 또는 인간 파트너(human partner)가 Superpowers 없이 무엇을 실행했고 어떤 결과가 나왔는지(해당 트랜스크립트 경로가 있다면 함께) 적으세요. 위 체크박스는 문제가 재발하지 않았음을 보여주는 트랜스크립트가 있을 때만 체크하세요.>

관여도(involvement)에 대한 증거는 아래에 있으며, 이는 인과관계(cause)를 단정하지 않습니다.

## What happened?

<문제 정의(Problem statement), 그리고 `path:line` 인용을 `transcript line <n>`으로 다시 작성한 트리아지 판정(triage verdict).>

## Steps to reproduce

1. <첫 번째 인간 프롬프트, 민감정보 제거됨(scrubbed)>
2. <문제로 이어진 턴들, 각 한 줄씩>
3. <관측된 현상(the observable)>

## Expected behavior

<문제 정의에서 가져온 기대 동작>

## Actual behavior

<트리아지 판정에서 가져온 실제 동작>

## Debug log or conversation transcript

Session id(s): <ids>. Delivered local archive: <path, redaction level <level> | none built>. Attached bundle: <no claim; attach only after approval>.
Superpowers involvement per the diagnosis report: <possible | likely>, with evidence at <transcript lines>. This report does not propose a fix.

---
Filed with the `diagnosing-superpowers` skill. The filing model and harness are under "Who's filing"; the diagnosed session's model, harness, versions, and plugins are in the Environment table.
