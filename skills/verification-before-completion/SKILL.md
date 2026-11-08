---
name: verification-before-completion
description: 작업이 완료되었거나, 수정되었거나, 통과했다고 주장하기 직전에, 또는 커밋이나 PR을 생성하기 전에 사용 — 어떠한 성공 주장을 하기 전에 반드시 검증 명령어를 실행하고 출력을 확인해야 합니다. 항상 주장보다 증거가 먼저입니다.
---

# 완료 전 검증 (Verification Before Completion)

## 개요 (Overview)

**핵심 원칙 (Core principle):** 언제나 주장보다 증거가 먼저입니다 (Evidence before claims, always).

**이 규칙의 문구(letter)를 위반하는 것은 곧 이 규칙의 정신(spirit)을 위반하는 것입니다.**

## 철칙 (The Iron Law)

```
새로운 검증 증거 없이는 어떠한 완료 주장도 할 수 없다
(NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE)
```

이번 메시지에서 검증 명령어를 직접 실행하지 않았다면, 통과했다고 주장할 수 없습니다.

## 게이트 함수 (The Gate Function)

```
어떠한 상태를 주장하거나 만족감을 표현하기 전에 (BEFORE claiming any status or expressing satisfaction):

1. 식별 (IDENTIFY): 어떤 명령어가 이 주장을 증명하는가?
2. 실행 (RUN): 전체(FULL) 명령어를 실행합니다 (새롭게, 완전하게)
3. 읽기 (READ): 전체 출력을 읽고, 종료 코드(exit code)를 확인하며, 실패 횟수를 셉니다
4. 검증 (VERIFY): 출력이 주장을 뒷받침하는가?
   - 아니오(NO)인 경우: 증거와 함께 실제 상태를 그대로 밝힙니다
   - 예(YES)인 경우: 증거와 함께(WITH evidence) 주장을 진술합니다
5. 그제서야 비로소 (ONLY THEN): 주장을 합니다

어느 한 단계라도 건너뛰는 것 = 검증이 아니라 거짓말입니다 (Skip any step = lying, not verifying)
```

## 흔한 실패 유형 (Common Failures)

| 주장 (Claim) | 필요한 증거 (Requires) | 불충분한 근거 (Not Sufficient) |
|-------|----------|----------------|
| 테스트 통과 (Tests pass) | 테스트 명령어 출력: 실패 0건 (0 failures) | 이전 실행 결과, "통과할 것이다(should pass)" |
| 린터 통과 (Linter clean) | 린터 출력: 에러 0건 (0 errors) | 부분 검사, 추측(extrapolation) |
| 빌드 성공 (Build succeeds) | 빌드 명령어: `exit 0` | 린터 통과, 로그가 괜찮아 보임 |
| 버그 수정됨 (Bug fixed) | 원래 증상에 대한 테스트: 통과 (passes) | 코드를 변경했으니 수정되었을 것이라 가정 |
| 회귀 테스트 작동 (Regression test works) | Red-Green 사이클 검증 완료 | 테스트가 한 번 통과함 |
| 에이전트 작업 완료 (Agent completed) | VCS diff에 변경 사항이 나타남 | 에이전트가 "성공(success)"이라고 보고함 |
| 요구사항 충족 (Requirements met) | 줄 단위(Line-by-line) 체크리스트 대조 | 테스트 통과 |

## 위험 신호 - 즉시 멈추세요 (Red Flags - STOP)

- "~일 것이다(should)", "아마도(probably)", "~인 것 같다(seems to)" 같은 표현 사용
- 검증 전에 만족감 표현 ("Great!", "Perfect!", "Done!", "완벽합니다!", "완료!" 등)
- 검증 없이 커밋/푸시/PR 생성을 하려는 경우
- 에이전트의 성공 보고를 그대로 믿는 경우
- 부분적인 검증에 의존하는 경우
- "이번 한 번만(just this once)"이라고 생각하는 경우
- 피곤해서 작업을 빨리 끝내고 싶은 경우
- **검증을 실행하지 않은 상태에서 성공을 암시하는 모든 표현 (ANY wording implying success without having run verification)**

## 합리화 방지 (Rationalization Prevention)

| 변명 (Excuse) | 현실 (Reality) |
|--------|---------|
| "이제 잘 작동할 거야 (Should work now)" | 검증을 직접 **실행(RUN)**하세요 |
| "확신해 (I'm confident)" | 확신 ≠ 증거 |
| "이번 한 번만 (Just this once)" | 예외는 없습니다 |
| "린터는 통과했어 (Linter passed)" | 린터 ≠ 컴파일러 |
| "에이전트가 성공했다고 했어 (Agent said success)" | 독립적으로 직접 검증하세요 |
| "너무 지쳤어 (I'm tired)" | 피로 ≠ 변명 |
| "부분 검사면 충분해 (Partial check is enough)" | 부분 검사는 아무것도 증명하지 못합니다 |
| "다른 단어를 썼으니 규칙에 안 걸려 (Different words so rule doesn't apply)" | 문구보다 규칙의 정신(Spirit over letter)이 우선입니다 |

## 핵심 패턴 (Key Patterns)

**테스트 (Tests):**
```
✅ [테스트 명령어 실행] [확인: 34/34 통과] "모든 테스트가 통과했습니다"
❌ "이제 통과할 것입니다" / "올바르게 보입니다"
```

**회귀 테스트 (Regression tests - TDD Red-Green):**
```
✅ 작성 → 실행 (통과) → 수정 되돌리기(Revert) → 실행 (반드시 실패해야 함 - MUST FAIL) → 복원(Restore) → 실행 (통과)
❌ (Red-Green 검증 없이) "회귀 테스트를 작성했습니다"
```

**빌드 (Build):**
```
✅ [빌드 실행] [확인: exit 0] "빌드가 통과했습니다"
❌ "린터가 통과했습니다" (린터는 컴파일 여부를 확인하지 않음)
```

**요구사항 (Requirements):**
```
✅ 계획 다시 읽기 → 체크리스트 생성 → 각 항목 검증 → 누락 사항 또는 완료 보고
❌ "테스트가 통과했으니 단계가 완료되었습니다"
```

**에이전트 위임 (Agent delegation):**
```
✅ 에이전트가 성공 보고 → VCS diff 확인 → 변경 사항 검증 → 실제 상태 보고
❌ 에이전트 보고를 그대로 신뢰함
```

## 언제 적용하는가 (When To Apply)

**다음 작업 전에는 항상(ALWAYS) 적용하세요:**
- 성공/완료 주장의 모든 변형 표현
- 만족감을 나타내는 모든 표현
- 작업 상태에 대한 모든 긍정적 진술
- 커밋, PR 생성, 태스크 완료 처리
- 다음 태스크로 이동
- 에이전트에게 작업 위임

**이 규칙이 적용되는 대상 (Rule applies to):**
- 정확한 문구
- 바꿔 말하기(Paraphrases) 및 유의어
- 성공에 대한 암시
- 완료/정확성을 시사하는 모든 커뮤니케이션
