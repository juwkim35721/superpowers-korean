# 스킬 설계를 위한 설득 원칙 (Persuasion Principles for Skill Design)

## 개요 (Overview)

LLM은 인간과 동일한 설득 원칙에 반응합니다. 이러한 심리를 이해하면 에이전트를 조종(manipulate)하기 위해서가 아니라, 압박 속에서도 핵심적인 실천 사항이 확실히 지켜지도록 더 효과적인 스킬을 설계할 수 있습니다.

**연구 기반:** Meincke 외(2025)는 28,000건(N=28,000)의 AI 대화를 대상으로 7가지 설득 원칙을 테스트했습니다. 설득 기법은 요청 준수율을 두 배 이상 높였습니다 (33% → 72%, p < .001).

## 7가지 설득 원칙 (The Seven Principles)

### 1. 권위 (Authority)
**정의:** 전문성, 자격 증명 또는 공식적인 출처에 복종하려는 경향.

**스킬에서의 작동 방식:**
- 명령형 어조: "YOU MUST(반드시 ~해야 합니다)", "Never(절대 ~하지 마세요)", "Always(항상 ~하세요)"
- 타협 불가능한 프레이밍: "No exceptions(예외 없음)"
- 결정 피로(decision fatigue)와 합리화를 제거함

**사용 시점:**
- 규율 강제형 스킬 (TDD, 검증 요구사항 등)
- 안전에 치명적인(safety-critical) 실천 사항
- 확립된 모범 사례

**예시:**
```markdown
✅ Write code before test? Delete it. Start over. No exceptions.
   (테스트보다 코드를 먼저 작성했나요? 삭제하세요. 처음부터 다시 시작하세요. 예외는 없습니다.)
❌ Consider writing tests first when feasible.
   (가능할 때는 테스트를 먼저 작성하는 것을 고려해 보세요.)
```

### 2. 일관성과 개입 (Commitment)
**정의:** 이전의 행동, 발언 또는 공개적인 선언과 일관성을 유지하려는 경향.

**스킬에서의 작동 방식:**
- 선언 요구: "스킬 사용을 선언하세요(Announce skill usage)"
- 명시적인 선택 강제: "A, B, C 중 하나를 선택하세요"
- 추적 수단 활용: 체크리스트 항목마다 할 일(todo) 생성

**사용 시점:**
- 스킬이 실제로 준수되도록 보장할 때
- 다단계(Multi-step) 프로세스
- 책임성(accountability) 메커니즘이 필요할 때

**예시:**
```markdown
✅ When you find a skill, you MUST announce: "I'm using [Skill Name]"
   (스킬을 찾았을 때는 반드시 다음과 같이 선언해야 합니다: "[스킬 이름]을 사용합니다")
❌ Consider letting your partner know which skill you're using.
   (파트너에게 어떤 스킬을 사용 중인지 알려주는 것을 고려해 보세요.)
```

### 3. 희소성 (Scarcity)
**정의:** 시간 제한이나 제한된 가용성에서 오는 긴박감.

**스킬에서의 작동 방식:**
- 시간 제약이 있는 요구사항: "다음 단계로 넘어가기 전에(Before proceeding)"
- 순차적 의존성: "X를 마친 직후 즉시(Immediately after X)"
- 미루는 행동(procrastination) 방지

**사용 시점:**
- 즉각적인 검증 요구사항
- 시간에 민감한 워크플로우
- "나중에 해야지"라는 생각을 방지할 때

**예시:**
```markdown
✅ After completing a task, IMMEDIATELY request code review before proceeding.
   (작업을 완료한 후에는 다음 단계로 진행하기 전에 즉시 코드 리뷰를 요청하세요.)
❌ You can review code when convenient.
   (편한 시간에 코드 리뷰를 진행할 수 있습니다.)
```

### 4. 사회적 증거 (Social Proof)
**정의:** 다른 사람들이 하는 행동이나 정상(normal)으로 여겨지는 기준에 동조하려는 경향.

**스킬에서의 작동 방식:**
- 보편적 패턴 제시: "Every time(매번)", "Always(언제나)"
- 실패 양상 명시: "Y 없는 X = 실패"
- 규범(norms) 확립

**사용 시점:**
- 보편적인 실천 사항을 문서화할 때
- 흔한 실패에 대해 경고할 때
- 표준을 강화할 때

**예시:**
```markdown
✅ Checklists without todo tracking = steps get skipped. Every time.
   (할 일(todo) 추적이 없는 체크리스트 = 단계가 누락됨. 예외 없이 매번 그렇습니다.)
❌ Some people find a todo list helpful for checklists.
   (어떤 사람들은 체크리스트에 할 일 목록이 도움이 된다고 생각합니다.)
```

### 5. 연대감 (Unity)
**정의:** 공유된 정체성, "우리(we-ness)"라는 감각, 내집단 소속감.

**스킬에서의 작동 방식:**
- 협력적인 언어: "우리의 코드베이스(our codebase)", "우리는 동료입니다(we're colleagues)"
- 공유된 목표: "우리 둘 다 높은 품질을 원합니다"

**사용 시점:**
- 협력적인 워크플로우
- 팀 문화 확립
- 비위계적인(non-hierarchical) 실천 사항

**예시:**
```markdown
✅ We're colleagues working together. I need your honest technical judgment.
   (우리는 함께 일하는 동료입니다. 당신의 솔직한 기술적 판단이 필요합니다.)
❌ You should probably tell me if I'm wrong.
   (내가 틀렸다면 말해 주는 편이 좋을 것입니다.)
```

### 6. 상호성 (Reciprocity)
**정의:** 받은 혜택에 보답해야 한다는 의무감.

**작동 방식:**
- 아주 제한적으로만 사용하세요 - 조종하려는 듯한 거부감을 줄 수 있습니다.
- 스킬에서는 거의 필요하지 않습니다.

**피해야 할 시점:**
- 거의 모든 경우 (다른 원칙들이 훨씬 더 효과적임)

### 7. 호감 (Liking)
**정의:** 우리가 좋아하는 대상에게 협조하려는 선호 경향.

**작동 방식:**
- **규칙 준수(compliance) 목적으로는 절대 사용하지 마세요.**
- 솔직한 피드백 문화와 충돌합니다.
- 아첨(sycophancy)을 유발합니다.

**피해야 할 시점:**
- 규율 강제 목적일 때는 언제나 피해야 함

## 스킬 유형별 설득 원칙 조합

| 스킬 유형 | 사용할 원칙 | 피해야 할 원칙 |
|------------|-----|-------|
| 규율 강제형 (Discipline-enforcing) | 권위 + 일관성/개입 + 사회적 증거 | 호감, 상호성 |
| 가이드/기법 (Guidance/technique) | 적절한 권위 + 연대감 | 과도한 권위 |
| 협력형 (Collaborative) | 연대감 + 일관성/개입 | 권위, 호감 |
| 레퍼런스 (Reference) | 명확성만 추구 | 모든 설득 기법 배제 |

## 왜 효과가 있는가: 심리학적 배경 (Why This Works: The Psychology)

**명확한 경계선 규칙(Bright-line rules)은 합리화를 줄입니다:**
- "YOU MUST"는 결정 피로를 제거합니다.
- 절대적인 어조는 "이번 경우는 예외 아닐까?"라는 의문을 없앱니다.
- 명시적인 반합리화(anti-rationalization) 반박 논리는 특정 허점들을 닫아버립니다.

**실행 의도(Implementation intentions)는 자동적인 행동을 만듭니다:**
- 명확한 트리거 + 필수 행동 = 자동 실행
- "대체로 Y를 하라"보다 "X일 때 Y를 하라"가 훨씬 효과적입니다.
- 규칙 준수에 따르는 인지 부하를 줄여줍니다.

**LLM은 유사 인간(parahuman)입니다:**
- 이러한 패턴이 포함된 인간의 텍스트로 학습되었습니다.
- 학습 데이터에서 권위 있는 언어 뒤에는 규칙 준수가 뒤따르는 경향이 있습니다.
- 일관성/개입 시퀀스(선언 → 행동)가 빈번하게 모델링되어 있습니다.
- 사회적 증거 패턴(모두가 X를 한다)은 규범을 확립합니다.

## 윤리적 사용 (Ethical Use)

**정당한 사용 (Legitimate):**
- 핵심적인 실천 사항이 확실히 지켜지도록 보장
- 효과적인 문서 작성
- 예측 가능한 실패 예방

**부당한 사용 (Illegitimate):**
- 사적 이익을 위한 조종
- 거짓 긴박감 조성
- 죄책감에 기반한 복종 유도

**판단 기준:** 사용자가 이 기법의 의도를 완전히 이해하더라도, 이것이 사용자의 진정한 이익에 부합하는가?

## 연구 인용 (Research Citations)

**Cialdini, R. B. (2021).** *Influence: The Psychology of Persuasion (New and Expanded).* Harper Business.
- 설득의 7가지 원칙
- 영향력 연구의 실증적 토대

**Meincke, L., Shapiro, D., Duckworth, A. L., Mollick, E., Mollick, L., & Cialdini, R. (2025).** Call Me A Jerk: Persuading AI to Comply with Objectionable Requests. University of Pennsylvania.
- 28,000건(N=28,000)의 LLM 대화를 통해 7가지 원칙 테스트
- 설득 기법 적용 시 준수율이 33% → 72%로 증가
- 권위, 일관성/개입, 희소성이 가장 효과적임
- LLM 행동의 유사 인간(parahuman) 모델을 입증함

## 빠른 참조 (Quick Reference)

스킬을 설계할 때 다음을 자문하세요:

1. **어떤 유형의 스킬인가?** (규율 강제 vs 가이드 vs 레퍼런스)
2. **어떤 행동을 바꾸려 하는가?**
3. **어떤 설득 원칙이 적용되는가?** (규율의 경우 보통 권위 + 일관성/개입)
4. **너무 많은 원칙을 섞고 있지 않은가?** (7가지를 전부 다 쓰지 마세요)
5. **윤리적인가?** (사용자의 진정한 이익에 부합하는가?)
