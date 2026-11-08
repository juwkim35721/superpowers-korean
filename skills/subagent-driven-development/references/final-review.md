# 최종 리뷰 및 마무리 (Final Review and Finish)

`superpowers:subagent-driven-development`의 일부입니다. 계획의 모든 태스크가 완료된 시점입니다.

브랜치 전체에 대한 최종 리뷰도 리뷰 패키지를 전달받습니다: 현재 실행 중인 작업 트리에서 `bash scripts/review-package PLAN_FILE MERGE_BASE HEAD`를 절대 경로로 실행하고(`MERGE_BASE` = 브랜치가 시작된 기준 커밋, 예: `git merge-base main HEAD`), 출력된 파일 경로를 최종 리뷰 디스패치에 포함하여 최종 리뷰어가 git 명령어로 브랜치 diff를 다시 추출하는 대신 단일 파일을 읽도록 하세요. `superpowers:requesting-code-review`의 [code-reviewer.md](../../requesting-code-review/code-reviewer.md)를 사용하여, 사용 가능한 가장 뛰어난 모델(SKILL.md의 Model Selection 참조)로 디스패치하세요. 원장(ledger)에 기록된 보류된 사소한 항목(`deferred-minor`)들과 판정 후 보류된(`parked`) 줄들을 리뷰어에게 가리켜 주어, 병합 전에 반드시 수정해야 할 항목들을 분류(triage)할 수 있게 하세요.

만약 브랜치 전체 최종 리뷰에서 지적 사항이 반환되면, 전체 지적 사항 목록을 담아 **단 하나의** 수정 서브에이전트를 디스패치하세요 — 지적 사항마다 수정 에이전트를 따로 띄우지 마세요. 지적 사항별 수정 에이전트들은 각각 컨텍스트를 새로 구축하고 테스트 스위트를 반복 실행합니다. 실제 세션에서 최종 리뷰의 수정 웨이브(fix wave) 하나가 이전의 모든 태스크를 합친 것보다 더 많은 비용을 소모한 사례가 있습니다.
수정이 완료되면 해당 수정 웨이브에 대해 범위 지정 재리뷰(scoped re-review)를 정확히 한 번 실행하세요 (현재 실행 중인 작업 트리에서 수정 범위에 대해 `bash scripts/review-package PLAN_FILE FIX_BASE HEAD`를 절대 경로로 실행하고 [re-review-prompt.md](../re-review-prompt.md)를 사용).
남아 있는 미해결 지적 사항들은 수정 루프의 차단기(breaker, [references/fix-loop.md](fix-loop.md) 참조)에서와 동일한 방식으로 판정(adjudicate)하세요: 판정과 함께 보류(park)하거나, 핵심적인(load-bearing) 항목들에 대해 결정을 내리고 원장에 기록하세요. 여기서도 실행 중인 계획을 멈추게 하는 네 가지 사유(SKILL.md 상단 참조)에 해당할 때만 멈춥니다. 두 번째 수정 웨이브는 없습니다 — 남아 있는 핵심 지적 사항들은 `finishing-a-development-branch`가 선택지를 제시할 때 인간 파트너(human partner)에게 전달됩니다.

### 마무리 (Finish)

무엇이든 삭제하기 전에, 원장(ledger)에서 `Ruling:`이 포함된 모든 줄 — 사전 점검(preflight) 판정, 보류된 지적 사항, 차단기 판정 등 전부 — 을 수집하여 당신의 최종 메시지 안의 "내가 내린 판정들(Rulings I made)" 섹션 아래에 판정을 내린 순서대로, 그리고 각 판정이 틀렸을 때의 비용과 함께 정리하세요. 이 목록은 하나도 빠짐없이 망라되어야 합니다: 원장에 판정이 기록되어 있다면 이 목록에도 반드시 포함되어야 합니다. 이 목록은 당신이 인간 파트너를 대신해 내린 결정들이 파트너에게 전달되는 유일한 통로입니다 — 파트너는 이를 읽고 당신이 잘못 판단한 부분을 재작업할 수 있습니다. 워크스페이스와 함께 사라져 버린 판정은 비밀리에 내려진 결정이나 마찬가지입니다.

브랜치 전체 최종 리뷰가 깨끗하게 통과되고 수정 사항들이 병합되면, 이 계획의 워크스페이스를 삭제하세요(`rm -rf <workspace>`) — 이제부터는 git 히스토리가 공식 기록입니다. 같은 레벨에 있는 다른 디렉토리들은 다른 계획의 소유이므로 절대 건드리지 말고 그대로 두세요.

마지막으로 `superpowers:finishing-a-development-branch`를 사용하세요.
