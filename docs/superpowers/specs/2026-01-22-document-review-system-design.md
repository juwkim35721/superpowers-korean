# 문서 리뷰 시스템 설계 (Document Review System Design)

## 개요

superpowers 워크플로우에 두 가지 새로운 리뷰 단계를 추가합니다:

1. **스펙 문서 리뷰 (Spec Document Review)** - 브레인스토밍 이후, 계획 작성(writing-plans) 이전
2. **계획 문서 리뷰 (Plan Document Review)** - 계획 작성(writing-plans) 이후, 구현 이전

두 단계 모두 구현 리뷰에서 사용하는 반복 루프 패턴을 따릅니다.

## 스펙 문서 리뷰어 (Spec Document Reviewer)

**목적:** 스펙이 완전하고 일관성이 있으며 구현 계획을 수립할 준비가 되었는지 검증합니다.

**위치:** `skills/brainstorming/spec-document-reviewer-prompt.md`

**검토 항목:**

| 카테고리 | 확인 대상 |
|----------|------------------|
| 완전성 (Completeness) | TODO, 플레이스홀더, "TBD", 미완성 섹션 |
| 커버리지 (Coverage) | 누락된 에러 처리, 엣지 케이스, 통합 지점 |
| 일관성 (Consistency) | 내부 모순, 상충되는 요구사항 |
| 명확성 (Clarity) | 모호한 요구사항 |
| YAGNI | 요청하지 않은 기능, 오버엔지니어링 |

**출력 형식:**
```
## Spec Review

**Status:** Approved | Issues Found

**Issues (if any):**
- [Section X]: [issue] - [why it matters]

**Recommendations (advisory):**
- [suggestions that don't block approval]
```

**리뷰 루프:** 이슈 발견 -> 브레인스토밍 에이전트가 수정 -> 재리뷰 -> 승인될 때까지 반복.

**디스패치 메커니즘:** `subagent_type: general-purpose`로 Task 도구를 사용합니다. 리뷰어 프롬프트 템플릿이 전체 프롬프트를 제공합니다. 브레인스토밍 스킬의 컨트롤러가 리뷰어를 디스패치합니다.

## 계획 문서 리뷰어 (Plan Document Reviewer)

**목적:** 계획이 완전하고 스펙과 일치하며 작업 분해(task decomposition)가 적절한지 검증합니다.

**위치:** `skills/writing-plans/plan-document-reviewer-prompt.md`

**검토 항목:**

| 카테고리 | 확인 대상 |
|----------|------------------|
| 완전성 (Completeness) | TODO, 플레이스홀더, 미완성 태스크 |
| 스펙 정합성 (Spec Alignment) | 계획이 스펙 요구사항을 충족하는지, 범위 확장(scope creep)이 없는지 |
| 작업 분해 (Task Decomposition) | 태스크의 원자성, 명확한 경계 |
| 태스크 문법 (Task Syntax) | 태스크 및 단계의 체크박스 문법 |
| 청크 크기 (Chunk Size) | 각 청크가 1000줄 미만인지 |

**청크(Chunk) 정의:** 청크는 계획 문서 내에서 `## Chunk N: <name>` 제목으로 구분되는 논리적 태스크 그룹입니다. writing-plans 스킬은 논리적 단계(예: "Foundation", "Core Features", "Integration")를 기준으로 이러한 경계를 생성합니다. 각 청크는 독립적으로 리뷰할 수 있을 만큼 자체 완결성을 갖추어야 합니다.

**스펙 정합성 검증:** 리뷰어는 다음 두 가지를 모두 전달받습니다:
1. 계획 문서(또는 현재 청크)
2. 참조용 스펙 문서 경로

리뷰어는 두 문서를 모두 읽고 요구사항 커버리지를 비교합니다.

**출력 형식:** 스펙 리뷰어와 동일하지만, 범위가 현재 청크로 한정됩니다.

**리뷰 프로세스 (청크 단위):**
1. writing-plans가 청크 N을 생성함
2. 컨트롤러가 청크 N의 내용과 스펙 경로를 함께 전달하여 plan-document-reviewer를 디스패치함
3. 리뷰어가 청크와 스펙을 읽고 판정 결과를 반환함
4. 이슈가 있는 경우: writing-plans 에이전트가 청크 N을 수정하고 2단계로 이동
5. 승인된 경우: 청크 N+1로 진행
6. 모든 청크가 승인될 때까지 반복

**디스패치 메커니즘:** 스펙 리뷰어와 동일 — `subagent_type: general-purpose`로 Task 도구 사용.

## 업데이트된 워크플로우

```
brainstorming -> spec -> SPEC REVIEW LOOP -> writing-plans -> plan -> PLAN REVIEW LOOP -> implementation
```

**스펙 리뷰 루프 (Spec Review Loop):**
1. 스펙 작성 완료
2. 리뷰어 디스패치
3. 이슈가 있는 경우: 수정 -> 2단계로 이동
4. 승인된 경우: 다음 단계로 진행

**계획 리뷰 루프 (Plan Review Loop):**
1. 청크 N 작성 완료
2. 청크 N에 대해 리뷰어 디스패치
3. 이슈가 있는 경우: 수정 -> 2단계로 이동
4. 승인된 경우: 다음 청크 또는 구현 단계로 진행

## 마크다운 태스크 문법 (Markdown Task Syntax)

태스크와 각 단계는 체크박스 문법을 사용합니다:

```markdown
- [ ] ### Task 1: Name

- [ ] **Step 1:** Description
  - File: path
  - Command: cmd
```

## 에러 처리 (Error Handling)

**리뷰 루프 종료:**
- 엄격한 반복 횟수 제한 없음 — 리뷰어가 승인할 때까지 루프를 계속 진행함
- 루프가 5회 반복을 초과하면 컨트롤러가 인간에게 이 사실을 알리고 지침을 구해야 함
- 인간은 다음 중 하나를 선택할 수 있음: 반복 계속하기, 알려진 이슈를 감수하고 승인하기, 또는 중단하기

**이견 처리:**
- 리뷰어는 자문 역할을 함 — 이슈를 표시하지만 강제로 차단하지는 않음
- 에이전트가 리뷰어의 피드백이 부정확하다고 판단하면 수정 내용에 그 이유를 설명해야 함
- 동일한 이슈에 대해 3회 반복 후에도 이견이 지속되면 인간에게 알림

**잘못된 형식의 리뷰어 출력:**
- 컨트롤러는 리뷰어 출력에 필수 필드(Status, 해당하는 경우 Issues)가 있는지 검증해야 함
- 형식이 잘못된 경우, 기대하는 형식에 대한 안내와 함께 리뷰어를 다시 디스패치함
- 잘못된 형식의 응답이 2회 반복되면 인간에게 알림

## 변경할 파일 (Files to Change)

**새 파일:**
- `skills/brainstorming/spec-document-reviewer-prompt.md`
- `skills/writing-plans/plan-document-reviewer-prompt.md`

**수정할 파일:**
- `skills/brainstorming/SKILL.md` - 스펙 작성 후 리뷰 루프 추가
- `skills/writing-plans/SKILL.md` - 청크 단위 리뷰 루프 추가, 태스크 문법 예시 업데이트
