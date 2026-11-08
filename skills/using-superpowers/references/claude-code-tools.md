# Claude Code 도구 참고사항 (Tool Notes)

Claude Code는 기준이 되는 레퍼런스 하네스(reference harness)입니다. 스킬들은 Claude Code의 용어(서브에이전트 디스패치에는 `Agent`, 할 일 목록(todos), `Skill`)를 기본으로 사용하여 작성되어 있습니다. 이 문서는 스킬의 기본 구조보다 더 저렴한 비용으로 Claude Code에서 계획을 실행할 수 있는 한 가지 방법을 다룹니다. 이는 인간 파트너(human partner)가 선택적으로 활성화(opt-in)하는 방식이며, 스킬이 요구하는 사항 자체를 바꾸지는 않습니다.

## 서브에이전트 기반 개발을 위한 저비용 오케스트레이션 (Cheaper orchestration for subagent-driven development)

`superpowers:subagent-driven-development` 실행에서 가장 비용이 많이 드는 자리는 컨트롤러 세션(controller session)입니다. 컨트롤러는 모든 디스패치 결과와 모든 보고서를 읽으며, 대개 세션에서 가장 성능이 뛰어난 고비용 모델에서 실행되기 때문입니다. Claude Code는 중첩 서브에이전트(nested subagents, 기본값은 메인 대화 아래 3단계까지이며 `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH`로 조정 가능)를 지원하므로, 전체 루프를 한 단계 아래 계층에서 실행할 수 있습니다.

인간 파트너(human partner)가 요청하거나 조율(coordination) 작업에 세션의 메인 모델을 쓰기에는 비용이 너무 비싸다고 말한 경우, 중간 등급(mid-tier) 모델로 **단 하나의 오케스트레이터 서브에이전트**를 디스패치하면서 계획 파일 경로와 함께 `superpowers:subagent-driven-development`를 처음부터 끝까지(end to end) 사용하라는 지시를 전달하세요. 그러면 해당 오케스트레이터가 스킬의 모델 선택(Model Selection) 규칙에 따라 자체적인 구현자(implementer)와 리뷰어(reviewer) 서브에이전트들을 디스패치합니다. 작업 공간(workspace)과 원장(ledger)은 디스크에 저장되므로 계층이 하나 추가되더라도 손실되는 정보는 없습니다. 오케스트레이터의 최종 메시지에는 반드시 "내가 내린 판정(Rulings I made)" 목록이 원문 그대로(verbatim) 포함되어야 합니다. 이 목록은 의사결정 내용이 인간 파트너(human partner)에게 전달되는 통로이므로, 요약하지 말고 그대로 전달(relay)하세요.

이 방식은 전체 계획을 실행할 때만 사용하세요. 단일 태스크 디스패치를 중첩하는 것은 아무런 이득도 없이 에이전트 자리(seat)만 하나 더 늘릴 뿐입니다.
