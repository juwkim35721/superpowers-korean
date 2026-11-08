# "Cannot Verify From Diff" 항목 처리하기 ("Cannot Verify From Diff" Items)

`superpowers:subagent-driven-development`의 일부입니다.

태스크 리뷰어가 "⚠️ Cannot verify from diff" 항목 — 변경되지 않은 코드에 존재하거나 여러 태스크에 걸쳐 있는 요구사항 — 을 보고할 수 있습니다. 이러한 항목들은 리뷰의 나머지 부분을 차단하지 않지만, 태스크를 완료로 표시하기 전에 당신이 직접 각 항목을 확인하여 해결해야 합니다: 당신은 리뷰어에게는 없는 전체 계획과 태스크 간(cross-task) 컨텍스트를 가지고 있기 때문입니다. 만약 확인 결과 해당 항목이 실제 누락(real gap)으로 판명되면, 이를 명세 리뷰 실패(failed spec review)로 취급하세요 — 다른 지적 사항들과 함께 수정 루프(fix loop)로 들어갑니다.
