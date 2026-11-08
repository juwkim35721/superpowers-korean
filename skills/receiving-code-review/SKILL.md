---
name: receiving-code-review
description: 코드 리뷰 피드백을 받았을 때, 제안을 구현하기 전에, 특히 피드백이 모호하거나 기술적으로 의문스러울 때 사용 — 보여주기식 동의(performative agreement)나 맹목적인 구현이 아니라 기술적 엄밀함과 검증을 요구합니다.
---

# 코드 리뷰 수용하기 (Code Review Reception)

## 개요 (Overview)

코드 리뷰에는 감정적인 연기(emotional performance)가 아니라 기술적 평가(technical evaluation)가 필요합니다.

**핵심 원칙 (Core principle):** 구현하기 전에 검증하세요. 단정하기 전에 질문하세요. 사회적 편안함보다 기술적 정확성이 우선입니다.

## 응답 패턴 (The Response Pattern)

```
코드 리뷰 피드백을 받았을 때 (WHEN receiving code review feedback):

1. 읽기 (READ): 즉각 반응하지 말고 피드백 전체를 끝까지 읽습니다
2. 이해 (UNDERSTAND): 요구사항을 자신의 말로 다시 정리합니다 (또는 질문합니다)
3. 검증 (VERIFY): 실제 코드베이스 상태와 대조하여 확인합니다
4. 평가 (EVALUATE): 이(THIS) 코드베이스에 기술적으로 타당한가?
5. 응답 (RESPOND): 기술적 수용 또는 근거 있는 반박(pushback)을 합니다
6. 구현 (IMPLEMENT): 한 번에 한 항목씩 구현하고, 각각 테스트합니다
```

## 금지된 응답 (Forbidden Responses)

**절대 금지 (NEVER):**
- "전적으로 맞습니다!" ("You're absolutely right!" — 지침 파일의 명시적 위반)
- "좋은 지적입니다!" / "훌륭한 피드백입니다!" ("Great point!" / "Excellent feedback!" — 보여주기식 반응)
- "지금 바로 구현하겠습니다" ("Let me implement that now" — 검증하기 전)

**대신 이렇게 하세요 (INSTEAD):**
- 기술적 요구사항을 다시 서술합니다
- 명확히 하기 위해 질문합니다
- 제안이 틀렸다면 기술적 근거를 들어 반박(push back)합니다
- 바로 작업을 시작합니다 (말보다 행동이 우선)

## 모호한 피드백 처리하기 (Handling Unclear Feedback)

```
어떤 항목이라도 불명확하다면 (IF any item is unclear):
  멈추세요(STOP) - 아직 아무것도 구현하지 마세요
  불명확한 항목에 대해 설명(clarification)을 요청하세요

이유(WHY): 항목들이 서로 연관되어 있을 수 있습니다. 부분적인 이해는 잘못된 구현으로 이어집니다.
```

**예시 (Example):**
```
인간 파트너(your human partner): "1~6번을 수정해 주세요"
당신은 1, 2, 3, 6번은 이해했지만 4, 5번은 불명확합니다.

❌ 잘못된 대응 (WRONG): 1, 2, 3, 6번을 먼저 구현하고 4, 5번은 나중에 질문함
✅ 올바른 대응 (RIGHT): "1, 2, 3, 6번 항목은 이해했습니다. 진행하기 전에 4번과 5번에 대해 추가 설명이 필요합니다."
```

## 출처별 처리 방법 (Source-Specific Handling)

### 인간 파트너로부터 받은 피드백 (From your human partner)
- **신뢰할 수 있음 (Trusted)** - 이해한 후 구현합니다
- 범위가 불명확하다면 **여전히 질문해야 합니다 (Still ask)**
- **보여주기식 동의 금지 (No performative agreement)**
- **바로 행동으로 옮기거나 (Skip to action)** 기술적 확인만 남깁니다

### 외부 리뷰어로부터 받은 피드백 (From External Reviewers)
```
구현하기 전에 (BEFORE implementing):
  1. 확인: 이(THIS) 코드베이스에 기술적으로 올바른가?
  2. 확인: 기존 기능을 망가뜨리지 않는가?
  3. 확인: 현재 구현이 그렇게 작성된 이유가 있는가?
  4. 확인: 모든 플랫폼/버전에서 작동하는가?
  5. 확인: 리뷰어가 전체 컨텍스트를 이해하고 있는가?

제안이 틀린 것 같다면 (IF suggestion seems wrong):
  기술적 근거를 들어 반박(push back)합니다

쉽게 검증할 수 없다면 (IF can't easily verify):
  그대로 말합니다: "[X] 없이는 이를 검증할 수 없습니다. [조사/질문/진행] 중 어떻게 할까요?"

인간 파트너의 이전 결정과 충돌한다면 (IF conflicts with your human partner's prior decisions):
  멈추고 인간 파트너와 먼저 논의합니다
```

**인간 파트너의 규칙 (your human partner's rule):** "외부 피드백 - 회의적인 시각을 유지하되, 주의 깊게 확인하라."

## "전문적인(Professional)" 기능에 대한 YAGNI 점검 (YAGNI Check for "Professional" Features)

```
리뷰어가 "제대로(properly) 구현할 것"을 제안한다면:
  코드베이스에서 실제로 사용되는지 grep으로 검색합니다

  사용되지 않는다면: "이 엔드포인트는 호출되지 않습니다. 제거할까요 (YAGNI)?"
  사용된다면: 그때 제대로 구현합니다
```

**인간 파트너의 규칙 (your human partner's rule):** "너와 리뷰어 모두 나에게 보고한다. 우리에게 필요 없는 기능이라면 추가하지 마라."

## 구현 순서 (Implementation Order)

```
여러 항목의 피드백인 경우 (FOR multi-item feedback):
  1. 불명확한 모든 사항을 먼저(FIRST) 명확히 합니다
  2. 그런 다음 아래 순서로 구현합니다:
     - 블로킹 이슈 (기능 손상, 보안)
     - 간단한 수정 (오타, import 구문)
     - 복잡한 수정 (리팩토링, 로직 변경)
  3. 각 수정 사항을 개별적으로 테스트합니다
  4. 회귀(regressions)가 없는지 검증합니다
```

## 언제 반박해야 하는가 (When To Push Back)

다음 경우에는 반박(push back)하세요:
- 제안이 기존 기능을 망가뜨릴 때
- 리뷰어가 전체 컨텍스트를 파악하지 못했을 때
- YAGNI를 위반할 때 (사용되지 않는 기능)
- 현재 기술 스택에 기술적으로 맞지 않을 때
- 레거시/호환성 이유가 존재할 때
- 인간 파트너의 아키텍처 결정과 충돌할 때

**반박하는 방법 (How to push back):**
- 방어적인 태도가 아니라 기술적인 근거를 사용하세요
- 구체적인 질문을 던지세요
- 작동 중인 테스트/코드를 참조하세요
- 아키텍처와 관련된 문제라면 인간 파트너를 참여시키세요

**대놓고 반박하기가 불편하다면:** 그 긴장감을 솔직히 언급한 뒤, 당신이 발견한 문제를 인간 파트너에게 말씀드리세요. 파트너는 당신의 정직함을 고마워할 것입니다.

## 올바른 피드백 수용하기 (Acknowledging Correct Feedback)

피드백이 실제로 맞을 때:
```
✅ "수정했습니다. [변경된 내용에 대한 간략한 설명]" ("Fixed. [Brief description of what changed]")
✅ "좋은 발견입니다 - [구체적인 이슈]. [위치]에서 수정했습니다." ("Good catch - [specific issue]. Fixed in [location].")
✅ [그냥 수정하고 코드로 보여줌]

❌ "전적으로 맞습니다!" ("You're absolutely right!")
❌ "좋은 지적입니다!" ("Great point!")
❌ "잡아주셔서 감사합니다!" ("Thanks for catching that!")
❌ "[무엇이든] 감사합니다" ("Thanks for [anything]")
❌ 그 어떤 감사 표현도 금지 (ANY gratitude expression)
```

**감사 인사를 하지 않는 이유:** 행동이 말해줍니다. 그냥 고치세요. 수정된 코드 자체가 당신이 피드백을 수용했음을 보여줍니다.

**"감사합니다(Thanks)"라고 쓰려는 자신을 발견한다면:** 즉시 지우세요. 대신 무엇을 수정했는지 서술하세요.

## 반박이 틀렸을 때 깔끔하게 정정하기 (Gracefully Correcting Your Pushback)

반박했으나 당신이 틀렸던 경우:
```
✅ "맞으셨습니다 - [X]를 확인해 보니 [Y]로 동작하네요. 지금 구현하겠습니다." ("You were right - I checked [X] and it does [Y]. Implementing now.")
✅ "검증해 보니 말씀하신 내용이 맞습니다. [이유] 때문에 제 초기 이해가 틀렸습니다. 수정하겠습니다." ("Verified this and you're correct. My initial understanding was wrong because [reason]. Fixing.")

❌ 구구절절한 사과 (Long apology)
❌ 왜 반박했었는지 변명하기 (Defending why you pushed back)
❌ 과도한 설명 (Over-explaining)
```

정정 사항을 사실대로 간결히 밝히고 다음으로 넘어가세요.

## 흔한 실수 (Common Mistakes)

| 실수 (Mistake) | 해결책 (Fix) |
|---------|-----|
| 보여주기식 동의 (Performative agreement) | 요구사항을 서술하거나 바로 행동으로 옮길 것 |
| 맹목적인 구현 (Blind implementation) | 먼저 코드베이스와 대조하여 검증할 것 |
| 테스트 없이 일괄 수정 (Batch without testing) | 한 번에 하나씩 수정하고 각각 테스트할 것 |
| 리뷰어가 무조건 맞다고 가정 (Assuming reviewer is right) | 기존 기능을 망가뜨리지 않는지 확인할 것 |
| 반박 회피 (Avoiding pushback) | 심리적 편안함보다 기술적 정확성이 우선임 |
| 부분적 구현 (Partial implementation) | 모든 항목을 먼저 명확히 한 뒤 시작할 것 |
| 검증할 수 없는데도 그냥 진행 (Can't verify, proceed anyway) | 한계를 밝히고 방향을 질문할 것 |

## 실제 예시 (Real Examples)

**보여주기식 동의 (나쁜 예 - Performative Agreement):**
```
Reviewer: "레거시 코드를 제거하세요"
❌ "전적으로 맞습니다! 지금 바로 제거하겠습니다..."
```

**기술적 검증 (좋은 예 - Technical Verification):**
```
Reviewer: "레거시 코드를 제거하세요"
✅ "확인 중... 빌드 타겟은 10.15+인데 이 API는 13+가 필요합니다. 하위 호환성을 위해 레거시 코드가 필요합니다. 현재 구현의 번들 ID가 잘못되어 있는데 — 이를 수정할까요, 아니면 13 미만 지원을 중단할까요?"
```

**YAGNI (좋은 예):**
```
Reviewer: "데이터베이스, 날짜 필터, CSV 내보내기를 포함한 적절한 메트릭 추적을 구현하세요"
✅ "코드베이스를 grep으로 확인했으나 이 엔드포인트를 호출하는 곳이 없습니다. 제거할까요(YAGNI)? 아니면 제가 놓친 사용처가 있나요?"
```

**모호한 항목 (좋은 예 - Unclear Item):**
```
your human partner: "1~6번 항목을 수정해 주세요"
당신은 1, 2, 3, 6번은 이해했지만 4, 5번은 불명확합니다.
✅ "1, 2, 3, 6번은 이해했습니다. 구현하기 전에 4번과 5번에 대한 추가 설명이 필요합니다."
```

## GitHub 스레드 답글 달기 (GitHub Thread Replies)

GitHub의 인라인 리뷰 코멘트에 답글을 달 때는 최상위 PR 코멘트가 아니라 해당 코멘트 스레드(`gh api repos/{owner}/{repo}/pulls/{pr}/comments/{id}/replies`)에 답글을 작성하세요.
