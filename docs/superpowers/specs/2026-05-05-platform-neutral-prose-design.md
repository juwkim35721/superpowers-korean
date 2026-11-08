# 플랫폼 중립적 문구 — Phase A 설계 (Platform-neutral prose — Phase A design)

## 배경

Superpowers는 여러 에이전트 런타임(Claude Code, Codex, Cursor, OpenCode, Copilot CLI, Gemini CLI)에 배포됩니다. 스킬 콘텐츠와 지원 문서들은 처음에 Claude Code용으로 작성되었기 때문에 어떤 런타임의 에이전트에도 해당하는 자리에 "Claude"라는 표현을 사용하고 있습니다. OpenAI의 벤더링된 포크(openai/plugins#217)에서는 일괄 재작성을 시도했다가 과거 출처 경로, 모델 이름, 플랫폼별 설치 지침까지 덮어쓰는 바람에 곳곳에서 명백한 오류를 범했습니다. 우리는 그러한 실수를 피하면서도, 정말로 부수적인 위치에 쓰인 플랫폼 중심적 문구만 제거하고자 합니다.

전체 작업은 참조 카테고리별로 단계(Phase)를 나누어 진행합니다. **이 스펙은 Phase A만 다룹니다:** 특정 플랫폼에 국한되지 않는 맥락에서 "Claude"를 언급하는 일반적인 3인칭 문구입니다. 이후 단계(설정 파일 참조, 마케팅 문구, 도구 이름 참조)는 여기서는 범위 외이며 각각의 고유한 스펙을 갖게 됩니다.

## 범위 내 (In scope)

다음 위치에서 일반적으로 "Claude"를 언급하는 문구:

- 활성 스킬 디렉터리 내의 `skills/*/SKILL.md` 및 지원 `.md` 파일들
- `skills/writing-skills/anthropic-best-practices.md`
- `README.md` (플랫폼 마케팅이 아니라 일반 문구로 언급된 곳만 해당)

추가로 신조어 용어 변경 1건: `skills/writing-skills/SKILL.md`에서 **Claude Search Optimization (CSO) → Skill Discovery Optimization (SDO)**으로 변경합니다.

## 범위 외 (Out of scope)

- **플랫폼/런타임 설명문** — "In Claude Code:", 설치 지침, 도구 매핑 참조. (Phase D 후보.)
- **설정 파일 참조** — CLAUDE.md, AGENTS.md, GEMINI.md 우선순위 목록 및 "프로젝트 규칙을 넣을 위치" 안내 문구. (Phase B.)
- **도구 이름 참조** — `Skill`, `Bash`, `Read`, `Task`, `TodoWrite`. 스킬들은 Claude Code의 도구 어휘로 작성되어 있으며, 기존의 `references/{codex,copilot,gemini}-tools.md` 파일들이 이를 매핑합니다. (이 스펙을 작성할 당시에는 이를 보류하거나 건너뛸 계획이었습니다. 결과적으로 Phase E에서 이를 수행하여 활성 스킬 전반의 도구 이름을 동작 중심 언어로 교체하고 동일한 어휘를 중심으로 플랫폼 도구 참조 문서들을 통합했습니다.)
- README의 **마케팅 문구** — "Superpowers for Claude Code", 플랫폼 이름이 붙은 설치 섹션들. (Phase C.)
- **과거 기록 아티팩트** — `docs/plans/*.md`, `docs/superpowers/specs/*.md`, `CREATION-LOG.md`. 이들은 날짜가 기록된 특정 시점의 문서들이며, 이를 다시 쓰는 것은 역사를 왜곡하는 것입니다.
- **모델 식별자** — Claude Haiku / Sonnet / Opus. 이들은 실제 제품 이름입니다.
- **파일 이름 / URL 참조** — `CLAUDE.md`, `claude.com`, `claude-plugin/`, `~/.claude/` 아래의 경로들.
- **`anthropic-best-practices.md` 파일 이름** — 내부 본문은 재작성하더라도 파일 이름은 출처를 따라 그대로 유지합니다.

## 교체 스타일 (Replacement style)

영어 원문에서 자연스럽게 읽히도록 다음 표현들을 적절히 혼용합니다:

- **2인칭 — "your agent"**: 스킬 작성자에게 *그들의* 런타임에 대해 이야기할 때
  - "your agent reads the description"
- **3인칭 — "the agent" / "agents" / "an agent"**: 시스템 동작을 일반적으로 설명할 때
  - "Future agents find your skills"
  - "Use words an agent would search for"
  - "Agents read SKILL.md only when the skill becomes relevant"

주변 문장에 어울리는 것을 선택하되, 어색한 표현을 감수하면서까지 일관성을 강제하지 마세요. 항상 "the agent"라고 하기보다는 자연스러울 때는 복수형("future agents", "agents read")을 사용하세요.

### "Claude"로 유지되는 예외 항목 (Carve-outs)

- 모델 이름: Claude Haiku, Claude Sonnet, Claude Opus
- 파일 이름 및 URL: `CLAUDE.md`, `claude.com`, `~/.claude/`
- 런타임 자체를 지칭하는 브랜드 플랫폼 이름 "Claude Code" (이후 단계에서 다룸)

### 신조어 용어 변경

- **Claude Search Optimization (CSO) → Skill Discovery Optimization (SDO)**
  - `skills/writing-skills/SKILL.md`에 섹션 제목 및 주변 본문으로 등장합니다. 제목, 약어, 그리고 파일 내의 모든 상호 참조를 변경합니다.

## 영향을 받는 파일

예외 항목을 제외하도록 필터링한 `grep`을 기준으로 한 대략적인 건수:

| 파일 | 일반 문구 언급 건수 |
|------|------------------------|
| `skills/writing-skills/SKILL.md` | ~12 (CSO 제목 + 본문 포함) |
| `skills/writing-skills/anthropic-best-practices.md` | ~30 |
| `skills/writing-skills/examples/CLAUDE_MD_TESTING.md` | ~1 — 파일 이름은 유지됨(CLAUDE.md 테스트 아티팩트이므로); "Variant C: Claude.AI Emphatic Style" 제목도 유지됨(특정 스타일을 지칭하는 레이블이므로) |
| `README.md` | ~1 |

최종 목록은 구현 중에 필터링된 grep을 다시 실행하여 확인합니다.

## 커밋 계획

순서대로 진행하는 네 개의 원자적(atomic) 커밋:

1. `skills/writing-skills/SKILL.md`에서 **CSO → SDO로 이름 변경**. 기계적이고 격리된 작업이므로 용어에 대해 마음이 바뀌면 쉽게 되돌릴 수 있습니다.
2. **활성 스킬 본문** — `anthropic-best-practices.md`를 제외한 `skills/*/SKILL.md` 및 지원 `.md` 전반에서 일반적인 "Claude" → "agent" 형태로 변경.
3. **`anthropic-best-practices.md` 본문** — 동일한 치환 규칙 적용. 이 파일은 외부 문서를 가져와 맞게 수정(vendored adaptation)한 것이므로 별도의 커밋으로 분리합니다. 변경 사항을 격리하면 향후 업스트림과 대조할 때 읽기가 더 쉬워집니다.
4. **README.md 본문** *(필터링 후 일반 문구 언급이 남아있는 경우에만)*. 비어 있으면 건너뜁니다.

각 커밋 메시지에는 단계("Phase A")와 작업 단위("rename CSO to SDO", "agent prose in active skills" 등)를 명시하여 커밋 시리즈 자체가 문서 역할을 하도록 합니다.

## 검증 (Verification)

각 커밋 이후:

- `grep -rn "Claude" <touched-paths>` — 남아있는 모든 일치 항목은 반드시 문서화된 예외 항목(모델 이름, 파일 이름, URL, "Claude Code" 플랫폼 이름, 과거 기록 아티팩트)에 속해야 합니다.
- 수정된 파일을 처음부터 끝까지 읽습니다 — 치환으로 인해 문장 흐름, 대명사 일치 또는 목록의 병렬 구조가 깨지지 않았어야 합니다.
- 실행할 테스트는 없습니다. 이는 순수 본문 수정 작업입니다.

최종 커밋 이후:

- 라이브 세션에서 수정된 각 스킬을 훑어보며 어색하게 읽히는 부분이 없는지 확인합니다.

## 비목표 (Non-goals)

- 동작, 구조, 제목(CSO→SDO 제외), 예제, 코드 블록 또는 YAML 프론트매터를 변경하지 않습니다.
- 새로운 섹션, 콜아웃 또는 호환성 참고 사항을 도입하지 않습니다.
- 편집하는 김에 치환 범위를 넘어 문구를 "개선"하려 하지 않습니다.
