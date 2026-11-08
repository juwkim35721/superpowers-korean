---
name: using-superpowers
description: 모든 대화를 시작할 때 사용 - 명확화 질문을 포함한 어떠한 응답보다도 먼저 스킬 호출을 요구하며, 스킬을 찾고 사용하는 방법을 확립함 (Use when starting any conversation - establishes how to find and use skills, requiring skill invocation before ANY response including clarifying questions)
---

<SUBAGENT-STOP>
특정 태스크를 수행하기 위해 서브에이전트(subagent)로 디스패치된 경우, 이 스킬은 무시하세요.
</SUBAGENT-STOP>

<EXTREMELY-IMPORTANT>
현재 수행하려는 작업에 어떤 스킬이 적용될 가능성이 단 1%라도 있다고 생각된다면, 무조건(ABSOLUTELY MUST) 해당 스킬을 호출해야 합니다.

만약 당신의 태스크에 적용되는 스킬이 있다면, 당신에게 선택권은 없습니다. 반드시 그 스킬을 사용해야 합니다.

이것은 협상의 대상이 아닙니다. 어떤 식으로도 합리화하여 피해 갈 수 없습니다.
</EXTREMELY-IMPORTANT>

## 규칙 (The Rule)

**어떠한 응답이나 행동을 하기 전에 관련되거나 요청된 스킬을 먼저 호출하세요** — 여기에는 명확화 질문(clarifying questions)하기, 코드베이스 탐색하기, 파일 확인하기가 모두 포함됩니다. 호출해 본 뒤 상황에 맞지 않는 것으로 판명되면 그때 사용하지 않아도 됩니다.

**계획 모드(plan mode)에 들어가기 전:** 아직 브레인스토밍을 하지 않았다면 `brainstorming` 스킬부터 먼저 호출하세요.

그런 다음 "[목적]을 위해 [스킬]을 사용합니다(Using [skill] to [purpose])"라고 선언하고 스킬의 지침을 정확히 따르세요. 스킬에 체크리스트가 있다면 항목마다 할 일(todo)을 생성하세요.

## 스킬 우선순위 (Skill Priority)

여러 스킬이 적용될 때는 프로세스 스킬(process skills)이 우선합니다 — 프로세스 스킬이 접근 방식을 설정하면, 그다음 구현 스킬(implementation skills: frontend-design 등)이 이를 실행에 옮깁니다. `brainstorming`과 `systematic-debugging`이 Superpowers에서 가장 흔히 쓰이는 프로세스 스킬이지만, 이 규칙은 다른 모든 스킬에도 동일하게 적용됩니다.

- "X를 만들자" → `superpowers:brainstorming`을 먼저 수행한 후, 구현 스킬들을 사용합니다.
- "이 버그 고쳐줘" → `superpowers:systematic-debugging`을 먼저 수행한 후, 도메인 스킬들을 사용합니다.

## 위험 신호 (Red Flags)

이런 생각이 든다면 즉시 멈추세요(STOP)—당신은 지금 합리화(rationalizing)하고 있습니다:

| 생각 (Thought) | 현실 (Reality) |
|---------|---------|
| "이건 그냥 간단한 질문인데" | 질문도 태스크입니다. 스킬을 확인하세요. |
| "우선 컨텍스트가 더 필요해" | 스킬 확인은 명확화 질문보다 **먼저** 와야 합니다. |
| "일단 코드베이스부터 탐색해 보자" | 스킬이 **어떻게** 탐색해야 하는지 알려줍니다. 먼저 확인하세요. |
| "git이나 파일만 빠르게 확인해 볼 수 있잖아" | 파일에는 대화의 맥락(context)이 없습니다. 스킬을 확인하세요. |
| "일단 정보부터 수집하자" | 스킬이 **어떻게** 정보를 수집해야 하는지 알려줍니다. |
| "이건 정식 스킬까지 필요하진 않아" | 스킬이 존재한다면 사용하세요. |
| "이 스킬 내용은 기억하고 있어" | 스킬은 계속 진화합니다. 현재 버전을 읽으세요. |
| "이건 태스크라고 할 것도 없지" | 행동 = 태스크입니다. 스킬을 확인하세요. |
| "이 작업에 스킬을 쓰는 건 과해(overkill)" | 단순한 일도 복잡해지기 마련입니다. 스킬을 사용하세요. |
| "일단 이 한 가지만 먼저 해놓고 보자" | 무엇이든 하기 **전에** 스킬부터 확인하세요. |
| "지금 작업이 생산적으로 잘 진행되는 느낌이야" | 규율 없는 행동은 시간을 낭비합니다. 스킬이 이를 방지합니다. |
| "그게 무슨 뜻인지는 알고 있어" | 개념을 아는 것 ≠ 스킬을 사용하는 것. 반드시 호출하세요. |

## 플랫폼별 적용 (Platform Adaptation)

사용 중인 하네스가 아래 목록에 있다면, 해당 레퍼런스 파일을 읽어 특별 지침을 확인하세요:

- Claude Code: `references/claude-code-tools.md`
- Codex: `references/codex-tools.md`
- Pi: `references/pi-tools.md`
- Antigravity: `references/antigravity-tools.md`
- Hermes Agent: `references/hermes-tools.md`
- Muse: `references/muse-tools.md`

## 사용자 지침 (User Instructions)

사용자 지침(CLAUDE.md, AGENTS.md, GEMINI.md 등 및 직접적인 요청)은 스킬보다 우선하며, 스킬은 기본 동작(default behavior)보다 우선합니다. 오직 당신의 인간 파트너(human partner)가 명시적으로 건너뛰라고 지시했을 때만 스킬 워크플로우나 지침을 건너뛰세요.
