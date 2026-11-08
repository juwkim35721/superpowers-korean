# 합리화에 맞서 스킬을 빈틈없이 방어하기 (Bulletproofing Skills Against Rationalization)

superpowers:writing-skills의 일부입니다. 베이스라인 실패가 규율(discipline) 실패일 때, 즉 에이전트가 규칙을 알면서도 압박 속에서 이를 건너뛸 때 사용합니다.

## 합리화에 맞서 스킬을 빈틈없이 방어하기

규율을 강제하는 스킬(예: TDD)은 합리화(rationalization)에 저항할 수 있어야 합니다. 에이전트는 영리해서 압박을 받으면 어떻게든 허점(loophole)을 찾아냅니다.

**적용 범위(Scope):** 이 툴킷은 규율 실패(에이전트가 규칙을 알면서도 압박 속에서 건너뛰는 경우)를 위한 것입니다. 출력 형태가 잘못되었거나 필수 요소를 누락하는 문제의 경우, 금지 조항 기반의 방어(bulletproofing)는 오히려 역효과를 냅니다. 그런 경우에는 '실패 유형에 맞는 작성 형식 선택하기(Match the Form to the Failure)'의 형식을 사용하세요.

**심리학적 참고사항:** 설득 기법이 **왜(WHY)** 작동하는지 이해하면 이를 체계적으로 적용할 수 있습니다. 권위(authority), 일관성/개입(commitment), 희소성(scarcity), 사회적 증거(social proof), 연대감(unity) 원칙에 대한 연구 기반(Cialdini, 2021; Meincke et al., 2025)은 이 디렉터리의 persuasion-principles.md를 참조하세요.

### 모든 허점을 명시적으로 차단하라 (Close Every Loophole Explicitly)

규칙만 단순히 선언하지 말고, 구체적인 우회 시도(workaround)들을 명시적으로 금지하세요:

<Bad>
```markdown
Write code before test? Delete it.
(테스트보다 코드를 먼저 작성했나요? 삭제하세요.)
```
</Bad>

<Good>
```markdown
Write code before test? Delete it. Start over.

**No exceptions:**
- Don't keep it as "reference"
- Don't "adapt" it while writing tests
- Don't look at it
- Delete means delete

(테스트보다 코드를 먼저 작성했나요? 삭제하세요. 처음부터 다시 시작하세요.

**예외는 없습니다:**
- "참고용"으로 남겨두지 마세요
- 테스트를 작성하면서 기존 코드를 "적당히 수정해 맞추지(adapt)" 마세요
- 쳐다보지도 마세요
- 삭제란 완전히 지우는 것을 의미합니다)
```
</Good>

### "규칙의 정신 대 문구(Spirit vs Letter)" 논쟁을 차단하라

초반부에 근본 원칙을 추가하세요:

```markdown
**Violating the letter of the rules is violating the spirit of the rules.**
(**규칙의 문구를 어기는 것은 곧 규칙의 정신을 어기는 것입니다.**)
```

이 한 문장이 "규칙의 정신(취지)은 따르고 있다"는 식의 합리화 부류를 통째로 차단합니다.

### 합리화 테이블 구축하기 (Build Rationalization Table)

베이스라인 테스팅(아래 테스팅 섹션 참조)에서 나온 합리화들을 포착하세요. 에이전트가 내놓는 모든 변명을 테이블에 넣습니다:

```markdown
| Excuse (변명) | Reality (현실) |
|--------|---------|
| "Too simple to test" ("테스트하기엔 너무 단순하다") | Simple code breaks. Test takes 30 seconds. (단순한 코드도 깨집니다. 테스트 작성에는 30초면 충분합니다.) |
| "I'll test after" ("나중에 테스트하겠다") | Tests passing immediately prove nothing. (작성하자마자 통과하는 테스트는 아무것도 증명하지 못합니다.) |
| "Tests after achieve same goals" ("나중에 테스트해도 똑같은 목적을 달성한다") | Tests-after = "what does this do?" Tests-first = "what should this do?" (사후 테스트 = "이 코드가 무엇을 하는가?" 사전 테스트 = "이 코드가 무엇을 해야 하는가?") |
```

### 위험 신호(Red Flags) 목록 만들기

에이전트가 합리화를 시작할 때 스스로 점검(self-check)하기 쉽게 만드세요:

```markdown
## Red Flags - STOP and Start Over (위험 신호 - 즉시 멈추고 다시 시작하세요)

- Code before test (테스트보다 코드 먼저 작성)
- "I already manually tested it" ("이미 수동으로 테스트했다")
- "Tests after achieve the same purpose" ("나중에 테스트해도 목적은 같다")
- "It's about spirit not ritual" ("중요한 건 본질이지 형식적인 절차가 아니다")
- "This is different because..." ("이번 경우는 다르다, 왜냐하면...")

**All of these mean: Delete code. Start over with TDD.**
(**이 모든 생각은 다음을 의미합니다: 코드를 삭제하세요. TDD로 처음부터 다시 시작하세요.**)
```

### 위반 징후를 반영하여 SDO 업데이트하기 (Update SDO for Violation Symptoms)

규칙을 막 위반하려는 순간의 징후(symptoms)를 `description`에 추가하세요:

```yaml
description: use when implementing any feature or bugfix, before writing implementation code
# (어떤 기능이나 버그 수정이든 구현 코드를 작성하기 전에 사용합니다)
```
