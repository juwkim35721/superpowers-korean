# 기존 원장에서 작업 재개하기 (Resuming From an Existing Ledger)

`superpowers:subagent-driven-development`의 일부입니다. 이 계획의 워크스페이스 내 `<workspace>/progress.md`에 이미 원장(ledger)이 존재하는 경우입니다.

원장의 첫 번째 줄에 현재 당신의 계획 파일 이름이 적혀 있다면, `Task <N>: complete` 줄이 있는 태스크들은 이미 완료(DONE)된 것입니다 — 다시 디스패치하지 말고, 완료 표시가 없는 첫 번째 태스크부터 재개하세요. 마지막 줄이 수정 라운드(fix round)로 끝나는 태스크는 루프 진행 중(mid-loop)인 상태입니다: 다음 라운드부터 수정 루프를 재개하세요. 만약 원장의 첫 번째 줄에 다른 계획 파일 이름이 적혀 있거나, 예전의 단일 경로인 `.superpowers/sdd/progress.md`에 떠돌이 원장이 남아 있다면 이는 다른 계획의 진행 기록입니다: 그대로 두고 당신의 원장을 새로 시작하세요.
