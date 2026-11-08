# 플랫폼 중립적 설정 파일 참조 — Phase B 설계 (Platform-neutral config-file references — Phase B design)

## 배경

Phase A(`2026-05-05-platform-neutral-prose-design.md` 참조)에서는 일반적인 3인칭 "Claude" 문구를 에이전트 중립적인 형태로 교체했습니다. 이번 단계에서는 다음 카테고리인 스킬 내부의 플랫폼별 지침 파일(CLAUDE.md, AGENTS.md, GEMINI.md) 참조를 다룹니다.

이 플러그인은 여러 하네스에서 실행되며, 각 하네스는 자체 지침 파일을 읽습니다. 스킬이 마치 CLAUDE.md가 유일한 파일인 것처럼 지칭하는 것은 Codex / Gemini CLI / OpenCode에서는 성립하지 않는 Claude Code 중심적인 가정입니다.

## 범위 내 (In scope)

활성 스킬 내의 특정 두 줄:

1. **`skills/writing-skills/SKILL.md:58`** — `Project-specific conventions (put in CLAUDE.md)`
2. **`skills/receiving-code-review/SKILL.md:30`** — `"You're absolutely right!" (explicit CLAUDE.md violation)`

## 범위 외 (Out of scope)

- **`skills/using-superpowers/SKILL.md:22, 26`** — 지침 우선순위 목록. 이 목록은 이미 세 가지(CLAUDE.md, GEMINI.md, AGENTS.md)를 모두 포괄적으로 명시하고 있으며, 이는 올바른 형태입니다: 해당 섹션은 다중 플랫폼 플러그인에서 *무엇이 사용자 지침으로 간주되는지*에 대해 실제적인 설명을 하고 있기 때문입니다. 변경할 필요가 없습니다.
- **과거 기록 / 예제 아티팩트**:
  - `skills/systematic-debugging/CREATION-LOG.md` — 출처 표시 경로(`~/.claude/CLAUDE.md`)는 역사적 사실입니다.
  - `skills/writing-skills/examples/CLAUDE_MD_TESTING.md` — 이 파일 전체는 CLAUDE.md 콘텐츠 변형들을 테스트한 실제 예제입니다. 파일 이름, 본문, 그리고 `testing-skills-with-subagents.md`에서 이를 가리키는 참조는 모두 그대로 유지합니다. 이를 일반화하면 예제의 취지가 무색해집니다.
- **플랫폼 도구 참조** — Phase D 후보 항목들:
  - `skills/using-superpowers/SKILL.md:40` (GEMINI.md에 관한 Gemini CLI 도구 매핑 참고 사항)
  - `skills/using-superpowers/references/gemini-tools.md` (`save_memory`가 GEMINI.md에 영구 저장됨)

## 치환 규칙 (Substitution rules)

범위 내의 각 줄에 대해 하나씩, 두 가지 개별 규칙을 적용합니다.

### 규칙 1: "프로젝트별 규칙을 넣을 위치"

`writing-skills/SKILL.md:58`:

- **변경 전:** `Project-specific conventions (put in CLAUDE.md)`
- **변경 후:** `Project-specific conventions (put in your instructions file)`

특정 파일 이름을 하나 고르는 대신 일반적인 구문을 사용합니다. 하네스마다 읽는 파일이 다르므로(CLAUDE.md, AGENTS.md, GEMINI.md 등) 스킬이 특정 파일을 단정해서는 안 됩니다. 각 플랫폼이 선호하는 파일 이름을 명시하기에 적절한 곳은 플랫폼 도구 참조 문서(`references/{codex,copilot,gemini}-tools.md`)입니다.

### 규칙 2: "(explicit CLAUDE.md violation)" 괄호 부연 설명

`receiving-code-review/SKILL.md:30`:

- **변경 전:** `"You're absolutely right!" (explicit CLAUDE.md violation)`
- **변경 후:** `"You're absolutely right!" (explicit instruction-file violation)`

이 괄호 설명은 실질적인 역할을 수행하고 있습니다 — 해당 문구가 단순히 문체상 좋지 않은 수준을 넘어, 많은 사용자가 자신의 지침 파일에 넣어두는 규칙을 정면으로 위반한다는 신호를 줍니다. "instruction file(지침 파일)"은 AGENTS.md / CLAUDE.md / GEMINI.md를 포괄하는 자연스러운 크로스 플랫폼 용어이며, 특정 파일 이름을 고르거나 의미를 "common(흔한)" 정도로 약화시키지 않고도 원래의 신호를 그대로 유지합니다.

## 커밋 계획

순서대로 진행하는 원자적(atomic) 커밋:

1. **`writing-skills/SKILL.md`** — "프로젝트 규칙을 넣을 위치" 줄에서 CLAUDE.md → "your instructions file"로 변경
2. **`receiving-code-review/SKILL.md`** — 위반 괄호 설명에서 CLAUDE.md → instruction-file로 변경
3. **플랫폼 도구 참조 문서** — 독자가 "your instructions file"을 실제 파일 이름으로 연결해 이해할 수 있도록 각 `references/{codex,copilot,gemini}-tools.md`에 플랫폼별 선호 지침 파일 이름(CLAUDE.md, AGENTS.md, GEMINI.md 등)을 추가합니다.

각 커밋 메시지에는 "Phase B"와 해당 작업 단위를 명시합니다.

## 검증 (Verification)

각 커밋 이후:

- 주변 단락을 읽어 문법과 의미가 여전히 자연스럽게 통하는지 확인합니다.
- `grep -n "CLAUDE\.md" <touched-file>` — 활성 본문에 남아있는 일치 항목이 없어야 합니다 (예외 항목은 이미 문서화됨).

두 커밋 모두 완료된 이후:

- `grep -rn "CLAUDE\.md" skills/`는 문서화된 예외 항목(CREATION-LOG, CLAUDE_MD_TESTING 및 이를 가리키는 참조, using-superpowers의 우선순위 목록)만 반환해야 합니다.

## 비목표 (Non-goals)

- `using-superpowers/SKILL.md`의 우선순위 목록 순서는 건드리지 않습니다. CLAUDE.md / GEMINI.md / AGENTS.md의 순서를 바꾸는 것은 치환이 아니라 미관상의 변경이며 이번 범위에 속하지 않습니다.
- `examples/CLAUDE_MD_TESTING.md`의 이름을 바꾸거나 내용을 변경하지 않습니다.
- Gemini CLI 전용 도구 참조(Phase D 후보)는 수정하지 않습니다.

## 구현 참고사항 (Implementation note)

여기에 작성된 Phase B는 세 개의 커밋과 Claude Code 외의 세 가지 플랫폼 도구 참조 문서를 다루었습니다. 실제 구현에서는 대칭성을 위해 한 걸음 더 나아갔습니다: 커밋 `8505703`에서 네 번째 참조 문서인 `references/claude-code-tools.md`가 추가되었으며, 이를 통해 Claude Code의 지침 파일 규칙과 도구 이름 목록도 주변 스킬 본문에 암묵적으로 녹아 있는 대신 다른 플랫폼들과 나란히 자리 잡게 되었습니다. 이 추가 사항은 본 스펙 작성 시점에는 예상하지 못했으나 스펙의 의도와 부합합니다.
