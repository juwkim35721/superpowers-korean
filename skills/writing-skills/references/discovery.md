# 탐색 최적화를 위한 SKILL.md 작성법 (Writing SKILL.md for Discovery)

superpowers:writing-skills의 일부입니다. 에이전트가 스킬을 찾고 읽을 수 있도록 SKILL.md 템플릿과 스킬의 이름 짓기, 설명 작성, 크기 조절 방법을 다룹니다.

## SKILL.md 구조

**프론트매터 (YAML Frontmatter):**
- 두 가지 필수 필드: `name`과 `description` (지원되는 모든 필드는 [agentskills.io/specification](https://agentskills.io/specification) 참조)
- 총 1024자 이하
- `name`: 영문자, 숫자, 하이픈(`-`)만 사용 (괄호나 특수문자 금지)
- `description`: 3인칭 시점으로, 무엇을 하는지가 아니라 **오직 언제 사용해야 하는지**만 기술
  - 트리거 조건에 집중하도록 "Use when..."(또는 구체적 사용 조건)으로 시작
  - 구체적인 증상, 상황, 컨텍스트 포함
  - **절대로 스킬의 프로세스나 워크플로우를 요약하지 말 것** (이유는 아래의 스킬 탐색 최적화 섹션 참조)
  - 가능한 한 500자 이내로 유지

```markdown
---
name: Skill-Name-With-Hyphens
description: Use when [구체적인 트리거 조건 및 증상]
---

# 스킬 이름 (Skill Name)

## 개요 (Overview)
이 스킬은 무엇인가? 1~2문장으로 요약한 핵심 원칙.

## 사용 시점 (When to Use)
[판단이 자명하지 않은 경우에만 작은 인라인 플로우차트 삽입]

증상(SYMPTOMS) 및 유스케이스를 나열한 불릿 리스트
사용하지 말아야 할 때 (When NOT to use)

## 핵심 패턴 (Core Pattern - 기법/패턴 스킬용)
적용 전/후(Before/after) 코드 비교

## 빠른 참조 (Quick Reference)
자주 쓰이는 작업을 빠르게 훑어볼 수 있는 테이블 또는 불릿 리스트

## 구현 (Implementation)
간단한 패턴을 위한 인라인 코드
방대한 레퍼런스나 재사용 가능한 도구를 위한 외부 파일 링크

## 흔한 실수 (Common Mistakes)
잘못되기 쉬운 부분 + 해결책

## 실제 적용 효과 (Real-World Impact - 선택 사항)
구체적인 결과 수치 및 사례
```


## 스킬 탐색 최적화 (Skill Discovery Optimization, SDO / CSO)

**탐색의 핵심:** 향후 에이전트가 여러분의 스킬을 **찾아낼(FIND)** 수 있어야 합니다.

### 1. 풍부한 Description 필드

**목적:** 에이전트는 주어진 작업에 어떤 스킬을 로드할지 결정하기 위해 `description`을 읽습니다. "지금 당장 이 스킬을 읽어야 할까?"라는 질문에 답이 되도록 작성하세요.

**형식:** 트리거 조건에 집중하도록 "Use when..."으로 시작합니다.

**핵심 주의사항: Description = 언제 사용하는가 (스킬이 무엇을 하는가가 아님)**

`description`에는 오직 트리거 조건만 기술해야 합니다. `description`에 스킬의 프로세스나 워크플로우를 요약하지 마세요.

**이것이 중요한 이유:** 테스팅 결과, `description`에 스킬의 워크플로우가 요약되어 있으면 에이전트가 스킬 전체 본문을 읽는 대신 `description`만 보고 행동해 버리는 현상이 밝혀졌습니다. 예를 들어 `description`에 "작업 간 코드 리뷰 수행(code review between tasks)"이라고 적어두었더니, 스킬 본문의 플로우차트에는 **두 번의 리뷰**(스펙 준수 리뷰 후 코드 품질 리뷰)가 명확히 나와 있었음에도 에이전트는 단 **한 번의 리뷰**만 수행했습니다.

이후 `description`에서 워크플로우 요약을 빼고 "독립적인 작업들로 구성된 구현 계획을 실행할 때 사용(Use when executing implementation plans with independent tasks)"으로 바꾸자, 에이전트는 플로우차트를 정확히 읽고 2단계 리뷰 프로세스를 따랐습니다.

**함정:** 워크플로우를 요약한 `description`은 에이전트가 질러가는 지름길(shortcut)을 만들어 줍니다. 그 결과 에이전트는 정작 스킬 본문 문서를 건너뛰게 됩니다.

```yaml
# ❌ 나쁜 예(BAD): 워크플로우를 요약함 - 에이전트가 스킬 본문을 읽는 대신 이 설명만 따를 수 있음
description: Use when executing plans - dispatches subagent per task with code review between tasks

# ❌ 나쁜 예(BAD): 프로세스 세부 사항이 너무 많음
description: Use for TDD - write test first, watch it fail, write minimal code, refactor

# ✅ 좋은 예(GOOD): 워크플로우 요약 없이 오직 트리거 조건만 명시
description: Use when executing implementation plans with independent tasks in the current session

# ✅ 좋은 예(GOOD): 트리거 조건만 명시
description: Use when implementing any feature or bugfix, before writing implementation code
```

**작성 내용:**
- 이 스킬이 적용되어야 함을 알리는 구체적인 트리거, 증상, 상황을 사용하세요.
- *특정 언어에 종속된 증상*(`setTimeout`, `sleep`)이 아니라 *문제 자체*(경쟁 상태(race conditions), 일관성 없는 동작)를 기술하세요.
- 스킬 자체가 특정 기술 전용이 아니라면 트리거는 기술 중립적(technology-agnostic)으로 유지하세요.
- 스킬이 특정 기술 전용이라면 트리거에 해당 기술을 명시적으로 드러내세요.
- 3인칭 시점으로 작성하세요 (시스템 프롬프트에 주입됩니다).
- **절대로 스킬의 프로세스나 워크플로우를 요약하지 마세요.**

```yaml
# ❌ 나쁜 예(BAD): 너무 추상적이고 모호하며, 언제 사용해야 하는지 포함하지 않음
description: For async testing

# ❌ 나쁜 예(BAD): 1인칭 시점 사용
description: I can help you with async tests when they're flaky

# ❌ 나쁜 예(BAD): 스킬은 특정 기술 전용이 아닌데 특정 기술을 언급함
description: Use when tests use setTimeout/sleep and are flaky

# ✅ 좋은 예(GOOD): "Use when"으로 시작하고, 문제를 설명하며, 워크플로우 요약이 없음
description: Use when tests have race conditions, timing dependencies, or pass/fail inconsistently

# ✅ 좋은 예(GOOD): 명시적인 트리거를 가진 특정 기술 전용 스킬
description: Use when using React Router and handling authentication redirects
```

### 2. 키워드 커버리지 (Keyword Coverage)

에이전트가 검색할 법한 단어들을 사용하세요:
- 에러 메시지: "Hook timed out", "ENOTEMPTY", "race condition"
- 증상: "flaky", "hanging", "zombie", "pollution"
- 유의어: "timeout/hang/freeze", "cleanup/teardown/afterEach"
- 도구: 실제 명령어, 라이브러리 이름, 파일 확장자/유형

### 3. 설명적인 이름 짓기 (Descriptive Naming)

**능동태, 동사 우선(verb-first)으로 작성하세요:**
- ✅ `skill-creation` 대신 `creating-skills`
- ✅ `async-test-helpers` 대신 `condition-based-waiting`

### 4. 토큰 효율성 (Token Efficiency - 매우 중요)

**문제:** 시작 가이드(getting-started) 및 자주 참조되는 스킬들은 **모든** 대화마다 로드됩니다. 토큰 하나하나가 소중합니다.

**목표 단어 수:**
- getting-started 워크플로우: 각각 150단어 미만
- 자주 로드되는 스킬: 총 200단어 미만
- 그 외 스킬: 500단어 미만 (마찬가지로 간결하게 작성)

**기법:**

**세부 사항은 도구의 도움말(`--help`)로 넘기세요:**
```bash
# ❌ 나쁜 예(BAD): SKILL.md에 모든 플래그를 문서화함
search-conversations supports --text, --both, --after DATE, --before DATE, --limit N

# ✅ 좋은 예(GOOD): --help를 참조하도록 안내
search-conversations supports multiple modes and filters. Run --help for details.
```

**상호 참조(Cross-references)를 활용하세요:**
```markdown
# ❌ 나쁜 예(BAD): 워크플로우 세부 사항을 반복해서 적음
When searching, dispatch subagent with template...
[20줄의 반복되는 지침]

# ✅ 좋은 예(GOOD): 다른 스킬을 참조함
Always use subagents (50-100x context savings). REQUIRED: Use [other-skill-name] for workflow.
```

**예제를 압축하세요:**
```markdown
# ❌ 나쁜 예(BAD): 장황한 예제 (42단어)
your human partner: "How did we handle authentication errors in React Router before?"
You: I'll search past conversations for React Router authentication patterns.
[Dispatch subagent with search query: "React Router authentication error handling 401"]

# ✅ 좋은 예(GOOD): 최소한의 예제 (20단어)
Partner: "How did we handle auth errors in React Router?"
You: Searching...
[Dispatch subagent → synthesis]
```

**중복을 제거하세요:**
- 상호 참조된 스킬에 이미 있는 내용을 반복하지 마세요.
- 명령어 자체만 봐도 자명한 내용을 굳이 설명하지 마세요.
- 동일한 패턴에 대해 여러 개의 예제를 넣지 마세요.

**검증:**
```bash
wc -w skills/path/SKILL.md
# getting-started 워크플로우: 각각 150단어 미만을 목표로 함
# 자주 로드되는 기타 스킬: 총 200단어 미만을 목표로 함
```

**수행하는 행동(what you DO)이나 핵심 통찰로 이름을 지으세요:**
- ✅ `condition-based-waiting` > `async-test-helpers`
- ✅ `skill-usage` 대신 `using-skills`
- ✅ `flatten-with-flags` > `data-structure-refactoring`
- ✅ `root-cause-tracing` > `debugging-techniques`

**프로세스 이름에는 동명사(-ing)가 잘 어울립니다:**
- `creating-skills`, `testing-skills`, `debugging-with-logs`
- 능동적이며 현재 취하고 있는 행동을 잘 묘사함

### 5. 다른 스킬 상호 참조하기 (Cross-Referencing Other Skills)

**다른 스킬을 참조하는 문서를 작성할 때:**

명시적인 필수 마커와 함께 스킬 이름만 사용하세요:
- ✅ 좋은 예: `**REQUIRED SUB-SKILL:** Use superpowers:test-driven-development`
- ✅ 좋은 예: `**REQUIRED BACKGROUND:** You MUST understand superpowers:systematic-debugging`
- ❌ 나쁜 예: `See skills/testing/test-driven-development` (필수인지 여부가 불명확함)
- ❌ 나쁜 예: `@skills/testing/test-driven-development/SKILL.md` (강제 로드되어 컨텍스트를 소모함)

**`@` 링크를 쓰지 않는 이유:** `@` 문법은 파일을 즉시 강제 로드하여, 실제로 필요하기도 전에 200k 이상의 컨텍스트를 소모해 버립니다.
