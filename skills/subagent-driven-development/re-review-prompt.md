# 범위 지정 재리뷰 프롬프트 템플릿 (Scoped Re-Review Prompt Template)

수정 라운드(fix round) 이후 재리뷰를 디스패치할 때 이 템플릿을 사용하세요. 재리뷰어는 이전 지적 사항들이 해결되었는지 검증하고, 수정 diff로 인해 새로 깨진 부분(new breakage)이 없는지 확인합니다. 이는 처음부터 다시 하는 전체 리뷰가 아닙니다 — 전체 리뷰는 이미 완료되었습니다.

**목적:** 이전 리뷰에서 제기된 각 지적 사항이 해결되었는지, 그리고 그 수정 자체가 다른 문제를 일으키지 않았는지 검증합니다.

```
Subagent (general-purpose):
  description: "Re-review Task N fix round R"
  model: [MODEL — 필수: SKILL.md의 모델 선택(Model Selection) 기준에 따라 선택하세요. 모델을 생략하면 세션에서 가장 비싼 모델을 소리 없이 상속받게 됩니다]
  prompt: |
    당신은 단일 태스크의 수정 라운드를 재리뷰하고 있습니다. 이전 리뷰에서 지적 사항들이 나왔고, 구현자가 이를 수정하려고 시도했습니다. 당신의 임무는 각 지적 사항에 대해 평결을 내리고 수정 diff를 검사하는 것뿐입니다 — 그 외의 작업은 하지 마세요.

    ## 태스크 (The Task)

    태스크 브리프를 읽으세요: [BRIEF_FILE]

    ## 검증 대상 지적 사항 (The Findings Under Verification)

    [FINDINGS]

    ## 수정 내역 (The Fix)

    구현자의 보고서를 읽으세요 (수정 보고서는 파일 끝에 덧붙여져 있습니다):
    [REPORT_FILE]

    **Fix base:** [FIX_BASE_SHA] (이전 리뷰가 보았던 head 커밋)
    **Head:** [HEAD_SHA]
    **Diff 파일:** [DIFF_FILE]

    diff 파일을 한 번 읽으세요 — 이 파일에는 수정 커밋 목록, 통계 요약(stat summary), 그리고 주변 컨텍스트가 포함된 수정 diff가 담겨 있습니다. git 명령어를 다시 실행하지 마세요.
    만약 diff 파일이 없다면 직접 diff를 가져오세요:
    `git diff --stat [FIX_BASE_SHA]..[HEAD_SHA]` 및
    `git diff [FIX_BASE_SHA]..[HEAD_SHA]`.

    이 체크아웃에서 당신의 리뷰는 읽기 전용(read-only)이어야 합니다. 작업 트리(working tree), 인덱스, HEAD, 또는 브랜치 상태를 어떠한 방식으로든 변경하지 마세요.

    ## 서브에이전트를 디스패치하지 마세요 (You Do Not Dispatch Subagents)

    이 리뷰의 모든 작업을 직접 수행하세요. diff의 일부를 리뷰하기 위해 서브에이전트를 생성하지 마시고, 두 번째 의견을 듣기 위해 또 다른 리뷰어를 절대 생성하지 마세요. 이 프로세스는 작업에 필요한 모든 리뷰 좌석을 이미 제공하고 있습니다. 당신이 직접 생성한 리뷰어는 전체 비용을 그대로 소모하며 그 좌석 중 하나를 중복 수행할 뿐이고, 그 평결은 아무런 효력도 갖지 못합니다. 만약 한 번에 다 보기엔 diff가 너무 크다고 느껴진다면, 직접 여러 패스(pass)로 나누어 리뷰하고 보고서에 그 사실을 명시하세요.

    ## 범위 (Scope)

    당신의 리뷰 범위는 지적 사항 목록과 수정 diff입니다. 모든 지적 사항에 대해 평결을 내리세요. 수정 자체로 인해 새로 유입된 문제가 없는지 수정 diff를 검사하세요. 수정 작업이 건드리지 않은 코드는 절대 다시 리뷰하지 마세요: 만약 수정 diff 바깥에서 문제를 발견하더라도 `Out-of-Scope Observations` 아래에만 보고하세요 — 이러한 사항은 이 태스크를 차단하지 않으며 수정 루프를 연장시키지도 않습니다. 광범위한 브랜치 전체 리뷰는 모든 태스크가 완료된 후에 수행됩니다.

    ## 테스트 (Tests)

    구현자는 수정된 코드를 커버하는 테스트들을 다시 실행하고 그 결과를 보고서 파일 끝에 덧붙였습니다. 보고서를 검증되지 않은 주장으로 취급하세요: 수정 보고서에 커버리지 테스트의 이름과 그 출력 결과가 명시되어 있는지 확인하고, 그 주장을 실제 diff와 대조하여 검증하세요. 보고서를 확인하기 위해 테스트 스위트를 다시 실행하지 마세요. 코드를 읽다가 기존 테스트 실행 결과로는 답을 얻을 수 없는 구체적인 의구심이 들 때만 테스트를 실행하세요 — 그리고 그때도 집중 테스트(focused test)만 실행해야 하며, 절대 패키지 전체 스위트를 돌리지 마세요.

    ## 출력 형식 (Output Format)

    당신의 최종 메시지는 보고서 그 자체여야 합니다: 서두 없이 첫 번째 지적 사항에 대한 평결로 바로 시작하세요. 모든 줄은 평결이거나, `file:line`이 포함된 지적 사항이거나, 당신이 수행한 확인 내역이어야 합니다 — 서문이나 진행 과정 설명은 일절 넣지 마세요.

    ### Finding Verdicts

    '검증 대상 지적 사항(The Findings Under Verification)'에 있는 각 항목에 대해 순서대로:
    - **[지적 사항 한 줄 요약]** — ADDRESSED | NOT ADDRESSED (`file:line` 증거 포함). 단순히 "시도했음(Attempted)"은 해결된(addressed) 것이 아닙니다: 지적된 구체적인 결함이 더 이상 존재하지 않아야 합니다.

    ### New Breakage in the Fix Diff

    수정 자체가 망가뜨렸거나 새로 유입시킨 문제들을 심각도(Critical/Important/Minor) 및 `file:line`과 함께 적으세요. 깨끗하다면 "None"이라고 적으세요.

    ### Out-of-Scope Observations

    수정 diff 바깥에서 발견한 이슈들입니다. 태스크 완료를 차단하지 않으며(Non-blocking), 컨트롤러가 최종 리뷰를 위해 원장(ledger)에 기록해 둡니다. 없다면 "None"이라고 적으세요.

    ### Verdict

    **Fix round:** [All findings addressed, no new Critical/Important breakage | Findings remain open] — 미해결 상태인 항목들을 나열하세요.
```

**플레이스홀더 설명 (Placeholders):**
- `[MODEL]` — 필수: SKILL.md의 모델 선택(Model Selection) 기준에 따른 리뷰어 모델; 작은 수정 diff에 대한 범위 지정 재리뷰에는 저렴하거나 중간 등급(cheap-to-mid tier)의 모델을 사용합니다
- `[BRIEF_FILE]` — 태스크 브리프 파일 (구현자가 작업할 때 사용한 것과 동일한 파일)
- `[FINDINGS]` — 이전 리뷰에서 나온 Critical/Important 지적 사항 및 명세 누락 사항들을 글자 그대로 복사하여 불릿(bullet)당 하나씩 나열한 목록
- `[REPORT_FILE]` — 구현자의 보고서 파일 (끝에 수정 보고서가 덧붙여진 상태)
- `[FIX_BASE_SHA]` — 이전 리뷰가 보았던 head 커밋
- `[HEAD_SHA]` — 현재 커밋
- `[DIFF_FILE]` — 현재 실행 중인 작업 트리에서 `bash scripts/review-package PLAN_FILE FIX_BASE HEAD`를 절대 경로로 실행했을 때 출력된 파일 경로

**재리뷰어 반환값:** 지적 사항별 평결(ADDRESSED / NOT ADDRESSED), 수정 diff 내의 새로운 결함(new breakage), 범위 외 관찰 사항(out-of-scope observations), 그리고 라운드 종합 평결(round verdict).
