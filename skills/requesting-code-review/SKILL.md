---
name: requesting-code-review
description: 태스크를 완료할 때, 주요 기능을 구현한 후, 또는 병합(merge) 전에 작업물이 요구사항을 충족하는지 검증하기 위해 사용
---

# 코드 리뷰 요청하기 (Requesting Code Review)

문제가 연쇄적으로 커지기(cascade) 전에 잡아낼 수 있도록 코드 리뷰어 서브에이전트(code reviewer subagent)를 디스패치합니다. 리뷰어에게는 평가를 위해 정교하게 구성된 컨텍스트만 전달하며 — 절대 현재 세션의 전체 히스토리를 넘기지 않습니다.

**핵심 원칙 (Core principle):** 일찍 리뷰하고, 자주 리뷰하세요 (Review early, review often).

## 언제 리뷰를 요청하는가 (When to Request Review)

**필수 (Mandatory):**
- 서브에이전트 주도 개발(subagent-driven development)에서 각 태스크 완료 후
- 주요 기능(major feature) 구현 완료 후
- `main` 브랜치에 병합하기 전

**선택 사항이지만 가치 있는 경우 (Optional but valuable):**
- 작업이 막혔을 때 (새로운 시각 확보)
- 리팩토링하기 전 (베이스라인 점검)
- 복잡한 버그를 수정한 후

## 요청 방법 (How to Request)

**1. git SHA 가져오기:**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # 또는: git merge-base origin/main HEAD
HEAD_SHA=$(git rev-parse HEAD)
```

**2. 코드 리뷰어 서브에이전트 디스패치:**

[code-reviewer.md](code-reviewer.md)의 템플릿을 채워 `general-purpose` 서브에이전트를 디스패치합니다.

**플레이스홀더 (Placeholders):**
- `{DESCRIPTION}` - 구현한 내용에 대한 간략한 요약
- `{PLAN_OR_REQUIREMENTS}` - 코드가 수행해야 하는 작업(계획 또는 요구사항)
- `{BASE_SHA}` - 시작 커밋
- `{HEAD_SHA}` - 종료 커밋

**3. 피드백에 따른 조치 (Act on feedback):**
- Critical 이슈는 즉시 수정합니다
- Important 이슈는 다음 단계로 넘어가기 전에 수정합니다
- Minor 이슈는 나중을 위해 기록해 둡니다
- 리뷰어가 틀렸다면 (근거를 들어) 반박(push back)합니다

## 예시 (Example)

```
[방금 Task 2 완료: 검증 함수 추가]

You: 다음으로 넘어가기 전에 코드 리뷰를 요청하겠습니다.

BASE_SHA=$(git log --oneline | grep "Task 1" | head -1 | cut -d' ' -f1)
HEAD_SHA=$(git rev-parse HEAD)

[코드 리뷰어 서브에이전트 디스패치]
  DESCRIPTION: 4가지 이슈 유형을 처리하는 verifyIndex() 및 repairIndex() 추가
  PLAN_OR_REQUIREMENTS: docs/superpowers/plans/deployment-plan.md의 Task 2
  BASE_SHA: a7981ec
  HEAD_SHA: 3df7661

[서브에이전트 반환 결과]:
  Strengths: 깔끔한 아키텍처, 실제 동작을 검증하는 테스트
  Issues:
    Important: 진행 상태 표시기(progress indicators) 누락
    Minor: 보고 간격에 매직 넘버(100) 사용
  Assessment: 진행 준비 완료 (Ready to proceed)

You: [진행 상태 표시기 수정]
[Task 3으로 계속 진행]
```

## 흔한 합리화 (Common Rationalizations)

| 변명 (Excuse) | 현실 (Reality) |
|--------|---------|
| "리뷰어를 디스패치하는 대신 내가 직접 diff를 리뷰할래" | 당신은 코디네이터(coordinator)입니다 — 인라인으로 diff를 직접 리뷰하면 작업을 계속 이끌어가는 데 필요한 컨텍스트 윈도우가 소모됩니다. 리뷰어 서브에이전트를 디스패치하세요. diff와 평가는 리뷰어의 컨텍스트에 남고, 당신에게는 발견 사항(findings)만 돌아옵니다. |
| "리뷰어가 변경 사항을 이해하려면 내 전체 세션 히스토리가 필요해" | 세션 히스토리가 아니라 정교하게 구성된 컨텍스트만 전달하세요. 그래야 리뷰어가 당신의 사고 과정이 아닌 실제 작업 결과물(work product)에 집중할 수 있습니다. |

## 위험 신호 (Red Flags)

**절대 금지 (Never):**
- "간단한 변경이니까"라며 리뷰 건너뛰기
- Critical 이슈 무시하기
- Important 이슈를 수정하지 않은 채 다음으로 넘어가기
- 타당한 기술적 피드백에 맞서 말싸움하기

**리뷰어가 틀렸다면 (If reviewer wrong):**
- 기술적 근거를 들어 반박(push back)하세요
- 정상 작동을 증명하는 코드/테스트를 제시하세요
- 명확한 설명(clarification)을 요청하세요

템플릿 참조: [code-reviewer.md](code-reviewer.md)
