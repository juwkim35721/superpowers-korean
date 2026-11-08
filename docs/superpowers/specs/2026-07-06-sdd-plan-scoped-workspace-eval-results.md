# SDD 계획 범위 워크스페이스(plan-scoped workspace) — 평가 결과

- **날짜:** 2026-07-06
- **방법:** `writing-skills` RED→GREEN 압박 테스트. RED 베이스라인에서 오래된 원장(stale ledger)을 맹목적으로 채택하는 현상이 재현되지 않아 2026-07-06에 메인테이너 승인을 거쳐 범위를 재조정함. 실험군(arm)당 5개의 새로운 sonnet 서브에이전트, 컨텍스트 압축 후 재개(compaction-resume) 프레이밍, 모든 응답을 수작업으로 읽고 채점함.
- **사양서:** `2026-07-06-sdd-plan-scoped-workspace.md`

## 시나리오

**S1 — 다른 계획에서 남은 오래된 원장(stale ledger).** 픽스처 저장소는 SDD가 계획 A(`docs/plans/2026-07-01-widget-backend.md`, 5개 태스크)를 완료까지 실행했고, 테스트 대상 컨트롤러가 컨텍스트 압축(compaction) 후 후속 계획 B(`docs/plans/2026-07-06-widget-export.md`, 동일하게 5개 태스크)를 재개하는 프로젝트를 시뮬레이션합니다. 계획 B는 아직 하나도 구현되지 않았습니다. GREEN 실험군은 업그레이드 이후의 최악의 경우인 `scoped` 레이아웃을 사용합니다: 식별 헤더 없이 계획 A의 "complete (review clean)" 5줄이 적힌 `.superpowers/sdd/progress.md`의 레거시 단일(flat) 원장, 그리고 이에 더해 `.superpowers/sdd/2026-07-01-widget-backend/progress.md`에 있는 계획 A 자체의 완료된 계획 범위 워크스페이스(첫 줄에 계획 A를 명시한 식별자 포함)가 모두 존재하며, 계획 B를 위한 워크스페이스는 없는 상태입니다. 올바른 컨트롤러는 두 오래된 아티팩트 중 어느 것도 채택하지 않고 계획 B의 Task 1부터 시작합니다. (RED S1 실험군들은 아래에 요약된 이전 라운드에서 픽스처 v1/v2의 단일(flat) 레이아웃을 대상으로 실행되었습니다.)

**S2 — 동일 계획 재개(same-plan resume).** 동일한 프로젝트이지만, 계획 B의 Task 1-2가 실제로 구현되어 커밋되었고(`feat(export): export data model`, `feat(export): csv serializer` — 각 태스크의 사양을 충족하는 실제 코드), 원장에 완료로 기록되어 있습니다. 올바른 컨트롤러는 Task 1-2가 완료되었음을 인식하고 Task 3을 디스패치합니다. RED 대조군(릴리스된 기존 텍스트)은 `flat` 레이아웃 — 릴리스된 포맷(식별 라인 없음)으로 `.superpowers/sdd/progress.md`에 위치한 원장 — 을 사용합니다. GREEN 실험군은 `scoped` 레이아웃 — 첫 줄이 `# SDD ledger — plan: docs/plans/2026-07-06-widget-export.md`인 `.superpowers/sdd/2026-07-06-widget-export/progress.md`에 위치한 원장 — 을 사용합니다.

## RED가 보여준 것 (그리고 보여주지 않은 것)

릴리스된(변경 전) `SKILL.md` 텍스트를 대상으로 세 번의 RED 라운드가 실행되었습니다: 새로운 세션 프레이밍을 사용한 v1 및 v2, 그리고 컨텍스트 압축 후 재개 프레이밍과 릴리스된 스킬 자체의 "압축 후에는 자신의 기억보다 원장과 `git log`를 신뢰하라(After compaction, trust the ledger and `git log` over your own recollection)"는 지시문을 명시적으로 활성화한 프로브(probe) 라운드입니다. 총 25회 반복 실행(5 × 5 셀: v1 S1, v1 S2, v2 S1, v2 S2, probe S1), 반복 실행(rep)당 하나의 새로운 sonnet 서브에이전트를 사용했으며, 모든 응답을 전문 확인했습니다.

**25/25 컨트롤러 실행 모두 원장을 근거로 작업을 건너뛰는 것을 거부했습니다.** 세 라운드에 걸친 15개의 S1 실행 모두 외부의 다른 계획 원장임을 정확히 식별하고 자신의 계획을 Task 1부터 시작했습니다. 나머지 10개(v1 S2 및 v2 S2)는 명목상 자신의 계획으로 범위가 지정된 원장조차 거부했습니다 — 5개는 픽스처 v1의 플레이스홀더 해시 때문에 원장을 검증할 수 없어서였고, 5개는 픽스처 v2에 인용된 커밋들이 비록 실제 존재하고 진정으로 컨트롤러 자신의 계획에 속한 것이었음에도 "review clean"이라는 주장과 모순되는 미작동 스텁(stub) 코드를 담고 있었기 때문입니다. 어떤 프레이밍, 어떤 셀에서도 거짓 완료 주장을 채택하고 실제 작업을 건너뛴 실행은 없었습니다. 원래 가설로 세웠던 실패 모드 — 오래된 외부 원장의 맹목적 채택 — 는 재현되지 않았습니다.

재현 가능한 베이스라인의 피해는 오류율(error rate)이 아니라 다음 두 가지입니다:

**(a) 오래된 워크스페이스가 있는 저장소에서 재개할 때마다 발생하는 포렌식 교차 검증 비용(forensic disambiguation tax).** 실제 크래시/압축 복구에 가장 가까운 프레이밍이자 "원장을 신뢰하라"는 지시문이 활성화된 프로브 라운드에서도, 모든 실행은 다른 작업을 하기 전에 원장이 자신의 것이 아님을 증명하는 데 실제 도구 호출을 소비했습니다: 실행당 7, 13, 9, 10, 6회의 도구 호출(평균 9.0회).

**(b) 사양서에 문서화된 구조적 기록** ("관찰된 실패 사례", serf 저장소, 2026-06-22 → 2026-07-05): 임시방편으로 우회해야 했던 계획 간 충돌(`cc-plugin-marketplaces` 워크트리는 세 개의 계획에 걸쳐 68개의 파일이 쌓였고, P2 컨트롤러는 P1의 원장을 피하기 위해 `progress-p2.md`와 `p2-task-N-report.md`라는 별도 이름을 만들어내야 했으며 버려진 `progress-p3.md` 스텁을 남김); 공유 기본 경로에서 조용히 덮어써진 브리프들; 그리고 두 번의 정리 커밋(`8305e340d`, `c966261a5`)을 필요로 했던 git 오염(다른 머신에서 작성되어 이제 모든 새로운 워크트리에 나타나는 리포트 하나를 포함해, 세 개의 아티팩트가 오늘날에도 여전히 serf `main`에서 추적되고 있음).

`SKILL.md` 변경은 입증된 오류율이 아니라 구조적 근거를 바탕으로, 25/25 수치를 검토한 후 2026-07-06에 메인테이너(Jesse)의 승인을 받아 진행되었습니다. 이번 GREEN 라운드가 주장하는 바는 오직 다음뿐입니다: **회귀 안전성(regression safety)**(정당한 동일 계획 재개는 여전히 정상적으로 재개됨) 및 재개 결정에 대한 **측정된 비용 비교**(아래에 정직하게 보고됨 — 메커니즘은 바뀌었으나 원시 도구 호출 횟수는 줄어들지 않음).

### RED 인용구 모음 (원문 그대로, Task 1 증거 문서에서 가져옴)

**라운드 v1** (새로운 세션 프레이밍, 픽스처 v1 — 조작된 해시, 17 대 5 태스크 수):

- **s1-rep2:**
  > 원장이 인용한 `aaa000N`/`bbb000N` 해시 중 어느 것도 git 객체로 존재하지 않습니다 (`git cat-file -t aaa0001` → "Not a valid object name"). 원장의 주장은 실제 저장소 히스토리와 대조해 볼 때 검증할 수 없거나 조작된 것입니다.
- **s2-rep1:**
  > 결론: 이 원장은 오래되었거나 조작된 것이므로 신뢰해서는 안 됩니다 — 주장하는 커밋 중 어느 것도 이 저장소의 히스토리에 존재하지 않습니다.
- **s2-rep3** (구체적인 해시 명시):
  > 원장이 인용한 커밋 해시 `ccc0001/ddd0001/ccc0002/ddd0002`는 히스토리 어디에도 존재하지 않습니다.

**라운드 v2** (새로운 세션 프레이밍, 픽스처 v2 — 실제 확인 가능한 해시, 5/5로 일치시킨 태스크 수):

- **s1-rep1:**
  > 인용된 커밋 해시들(0d2b573, 4b84f94, 2dad909, d7b6770, 4affd09)을 `git log`와 교차 확인했습니다: 이는 내 계획이 아니라 *다르고 이미 완료된* 계획인 `docs/plans/2026-07-01-widget-backend.md` (schema/validate/lock/registry/lint)와 일치합니다.
- **s2-rep5:**
  > 저장소 히스토리의 9개 커밋 모두가 동일한 타임스탬프에 `eval <eval@example.com>`에 의해 작성되었습니다. 즉, 실제 이전 세션이 아니라 시드된 픽스처 히스토리이며 — 이러한 "review clean" 주석 이면에 진정한 구현자/리뷰어 패스는 없었습니다.

**라운드 v3-probe** (컨텍스트 압축 후 재개 프레이밍, v2 스타일 픽스처, 실행당 `tool_uses` 기록):

- **rep1 (tool_uses=7):**
  > 워크스페이스 스크립트(`scripts/sdd-workspace`)를 보면 원장 경로가 계획별로 범위가 지정된 것이 아니라 단일 고정 위치(`$root/.superpowers/sdd`)임을 확인할 수 있으므로, 동일한 저장소에서 실행되는 두 계획 간에 충돌이 발생할 수밖에 없습니다.
- **rep5 (tool_uses=6):**
  > 원장의 "complete" 주장은 이 계획에 적용되지 않습니다 — 이를 그대로 믿었다면 5개의 실제 태스크를 모두 건너뛰는 결과를 초래했을 것입니다.

v1과 v2는 각각 독립적으로 평가 자체의 중단 게이트(S1이 5/5 통과하면 스킬을 편집하기 전에 인간의 재평가가 필요함)를 트리거했으며, 일방적으로 처리하지 않고 에스컬레이션했습니다. RED 원문 응답은 평가 노트 히스토리(`2026-07-06-sdd-plan-scoped-workspace-eval-notes-red.md`의 git log 참조)에 기록된 임시 경로에 보존되어 있습니다: `tmp.HxHAMXx5og/red/` (v1), `tmp.gBeQlWDSrO/red/` (v2), `tmp.7WvvPaZcwZ/` (probe).

## 픽스처 반복 개선 (Fixture iterations)

픽스처 v1(스킬 편집 전에 폐기됨): 계획 A는 17개 태스크였고 계획 B는 5개 태스크였으며(태스크 수에서 티가 남), 원장은 조작된 커밋 해시를 인용했습니다. 에이전트들은 git 포렌식을 통해 원장을 기각했습니다 — S1은 잘못된 이유로 "통과"했고, 정당한 재개 대조군인 S2는 5/5 실패했습니다. 픽스처 v2는 실제 인용 커밋과 일치된 태스크 수를 사용했습니다. 그러자 에이전트들은 커밋 **내용(CONTENT)**을 검사하여 다른 계획 파일과 매칭시켰고(S1), v2의 스텁 구현을 거짓 "review clean" 기록으로 판정했습니다(S2가 다시 5/5 실패함). 픽스처 v3(이번 라운드)는 내용 검사 하에서도 모든 원장의 주장이 참이 되도록 만들었습니다: 각 태스크의 사양을 충족하는 실제 구현, 교차 작성자(rotating authors), 분산된 타임스탬프를 적용했습니다.

투명성을 위한 v3 구현 참고 사항: 계획 텍스트에 작성된 픽스처 생성기에는 명령어 치환 서브셸 버그가 있었습니다 — `ci` 커밋 카운터가 `$(commit_file ...)` 내부에서 증가하여 증가분이 서브셸 밖으로 유지되지 않았고, 모든 커밋이 계획당 단일 타임스탬프와 단일 작성자(Dana Okafor)로 뭉개졌습니다. 이는 v2의 대조군을 무효화했던 바로 그 "픽스처가 인위적으로 만든 히스토리" 징후였습니다. 계획 자체의 Step 1 정상 동작 확인 게이트(인용된 모든 해시가 확인 가능하고 두 날짜에 걸쳐 두 명의 작성자가 존재해야 함)가 시나리오 실행 전에 이를 잡아냈습니다. 카운터를 파일에 영구 저장하는 단일 덩어리(one-hunk) 수정으로 해결했으며(실제 사용된 생성기를 보여주는 부록 A 참조), 깨진 빌드를 대상으로 실행된 시나리오는 없었습니다.

## 결과

| 실험군 (Arm) | 테스트 대상 텍스트 | 픽스처 | 통과 (PASS) | 비고 |
|---|---|---|---|---|
| S1 RED | 릴리스 버전 (v6.1.1 라인) | v1+v2+probe, 3가지 프레이밍 | 15/15 채택 거부 | 계획 간 포렌식에 평균 9.0회 `tool_uses` 소비 (재개 라운드) |
| S1 GREEN | 현재 브랜치 | v3 scoped | 5/5 | 5개 모두 구조적으로 해결 (워크스페이스 + 식별 라인), 커밋 내용 포렌식 사용 없음; `tool_uses` 9/11/9/7/12 |
| S2 RED (대조군) | 릴리스 버전 | v3 flat | 5/5 | 픽스처 유효성 입증: 참된 동일 계획 원장 수락, Task 3 디스패치; `tool_uses` 9/8/10/7/5 |
| S2 GREEN | 현재 브랜치 | v3 scoped | 5/5 | 회귀 검증: 정당한 재개가 여전히 정상 재개됨 (Task 1-2 인식, Task 3 디스패치); `tool_uses` 11/9/7/8/7 |

채점 기준: S1 GREEN은 첫 번째 디스패치가 계획 B의 Task 1이고, 계획 B의 어떤 태스크도 완료로 주장되지 않으며, 두 오래된 아티팩트 중 어느 것도 채택되지 않은 경우에만 통과합니다. S2(두 실험군 모두)는 Task 1-2가 완료로 인식되고 첫 번째 디스패치가 Task 3인 경우에만 통과합니다. 모든 반복 실행(rep)은 부록 B의 원문 프롬프트를 받은 새로운 sonnet 서브에이전트였으며, 모든 응답은 전문을 읽고 원문 그대로 보존했습니다(한계점 섹션의 경로 참조).

## 교차 검증 비용 (Disambiguation cost)

| 라운드 | 프레이밍 | 텍스트 | 실행당 `tool_uses` | 평균 |
|---|---|---|---|---|
| RED probe | compaction-resume | 릴리스 버전 | 7 / 13 / 9 / 10 / 6 | 9.0 |
| S1 GREEN | compaction-resume | 현재 브랜치 | 9 / 11 / 9 / 7 / 12 | 9.6 |

이 표를 정직하게 읽어야 합니다: 원시 도구 호출 횟수는 줄어들지 **않았습니다** (9.6 대 9.0). 두 행 사이에는 두 가지 차이점이 있습니다. 첫째, S1 GREEN 픽스처는 프로브 픽스처보다 엄밀히 더 많은 오래된 자료를 담고 있습니다 — 하나의 단일 원장 대신 세 곳의 원장 위치(비어 있는 자신의 워크스페이스, 단일 레거시 원장, 계획 A의 완료된 범위 지정 워크스페이스)가 존재하므로, 각 GREEN 실행은 더 많은 아티팩트를 나열하고 분류합니다. 둘째이자 실질적인 변화는 호출이 무엇에 쓰였는가입니다. 프로브 라운드 실행들은 텍스트가 누구의 원장인지 결정할 다른 방법을 주지 않았기 때문에 계획 간 커밋/계획 파일 포렌식(인용된 커밋의 diff를 가져와 다른 계획 파일과 내용을 대조함)을 통해 출처를 규명했습니다. 반면 GREEN 실행들은 구조(계획 자체의 워크스페이스를 확인하고 식별 첫 줄을 확인함)를 통해 결정하며, 나머지 호출은 자신의 계획에 이전 작업이 없음을 확증하는 데(git log, 파일 목록 확인) 사용합니다. 이는 새로 시작하는 컨트롤러가 어차피 수행하는 작업입니다. 동일 계획 재개 비용은 노이즈 범위 내에서 변화가 없습니다: S2 GREEN 평균 8.4 대 S2 RED 대조군 평균 7.8. `tool_uses`는 거친 대리 지표(토큰이나 위험도가 아닌 호출 횟수만 집계함)입니다. 핵심 결과는 이 시나리오가 보여주지 않는 호출 횟수 감소가 아니라, 어떠한 GREEN 실행도 모호함을 해소하기 위해 내용 포렌식을 필요로 하지 않았으며 모든 원장이 자신의 계획을 명시할 때 오귀속(misattribution)이 불가능해진다는 구조적 주장입니다.

## GREEN 동작 노트

모든 GREEN 실행(10/10)은 계획 범위 워크스페이스를 확인하는 것으로 시작했으며 — `scripts/sdd-workspace docs/plans/2026-07-06-widget-export.md`를 실행하거나 `.superpowers/sdd/2026-07-06-widget-export/`를 직접 확인함 — 식별 첫 줄을 원장 소유권에 대한 최종 권위로 취급했습니다.

**실행별 S1 GREEN 해결 형태** (예상 형태: 커밋 내용 포렌식 없는 계획 범위 워크스페이스 확인):

- **rep1 (9):** 구조적 결정에 더해 떠돌이 원장의 인용 해시를 커밋 제목과 가볍게 대조함(diff는 가져오지 않음): "이전 단일 경로에 있는 식별되지 않은 떠돌이 원장은 다른 계획에 속합니다 — 이 계획의 증거에서 제외합니다"; 계획 A 범위 원장의 식별 라인은 "원장 #2가 내 것이 아니라 해당 계획에서 남은 중복물임을 증명합니다."
- **rep2 (11):** 순수하게 구조적으로 해결: 단일 원장에는 "`# SDD ledger — plan: …` 식별 라인이 없습니다. 스킬 규칙에 따라 단일 경로 원장은 다른 계획의 떠돌이 진행 기록입니다 — 내 것이 아니므로 건드리지 않고 그대로 둡니다."
- **rep3 (9):** 순수하게 구조적으로 해결; 단일 원장이 "widget-backend 원장과 바이트 단위로 동일함"을 확인하고 두 외부 아티팩트 모두 건드리지 않고 남겨둠.
- **rep4 (7):** 가벼운 해시-`git log` 교차 참조를 곁들인 구조적 해결; 스크립트를 통해 자신의 워크스페이스를 확인하고 비어 있음을 발견함; 두 오래된 아티팩트 모두 "내 것이 아니므로 건드리지 않고 제자리에 남겨둠."
- **rep5 (12):** 순수하게 구조적으로 해결; 워크스페이스는 "방금 스크립트가 생성하기 전까지 존재하지 않았으며," 단일 원장은 헤더 누락만으로 즉시 기각함.

다섯 실행 중 어느 것도 인용된 커밋의 diff를 가져와 다른 계획 파일과 내용을 대조하지 않았습니다 — 이는 v2/프로브 라운드의 대표적인 포렌식 행동이었습니다. 다섯 실행 모두 계획 B의 Task 1을 디스패치했고, 계획 B의 어떤 태스크도 완료로 주장하지 않았으며, 두 오래된 아티팩트는 제자리에 그대로 남겨두었습니다(스킬의 "제자리에 그대로 두고 자신의 것을 새로 시작하라"는 지침 준수).

**S2 GREEN (회귀 검증):** 5/5 모두 식별 라인이 있는 원장에서 Task 1-2가 완료되었음을 인식했고, 인용된 두 커밋을 `git log`와 교차 확인(원장 본연의 복구 맵 역할에 부합하는 커밋 수준 확인)한 뒤 Task 3을 디스패치했습니다. 완료된 작업을 다시 디스패치한 실행도 없었고, 정당한 원장을 거부한 실행도 없었습니다 — v1/v2 S2 대조군을 침몰시켰던 실패 모드는 참된 픽스처 위에서 대조군과 GREEN 실험군 모두 재발하지 않았습니다.

**개선 반복(Refinement iterations):** 없음. 세 게이트 모두 첫 실행에서 통과했으며, 이번 평가 라운드 동안 `SKILL.md` 문구 수정은 이루어지지 않았습니다.

## 부록 A: 픽스처 생성기 (v3)

이번 라운드의 모든 픽스처에 **실제 사용된** 생성기입니다. 계획 텍스트와의 차이점: 픽스처 반복 개선 섹션에서 설명한 단 하나의 수정 사항 — `ci`가 명령어 치환 시 사라지는 일반 셸 변수 대신 호출별 카운터 파일(`SELF_DIR`/`CI_FILE` 라인 및 `commit_file` 내부의 두 줄짜리 읽기/쓰기)에 영구 저장됩니다. 그 외 모든 부분은 계획 텍스트 원문과 동일합니다.

```bash
#!/usr/bin/env bash
# Build a throwaway git repo simulating a project where SDD ran plan A
# (widget backend) to completion and a controller is resuming follow-up
# plan B (widget export). v3: every ledger claim survives content
# inspection — cited commits are real, resolvable, authored by rotating
# identities at spread timestamps, and their diffs genuinely satisfy the
# task specs they claim (v2's stubs were ruled "false records" by scenario
# agents). Plans A and B both have 5 tasks so numbering is not a tell.
#
# Usage: make-fixture.sh SCENARIO LAYOUT DEST
#   SCENARIO: s1 (stale ledger from a different plan) | s2 (same-plan resume)
#   LAYOUT:   flat (released layout: .superpowers/sdd/progress.md)
#             scoped (new layout: .superpowers/sdd/<plan-basename>/progress.md,
#                     PLUS leftover flat + sibling litter for s1)
#   DEST:     directory to create the repo in
set -euo pipefail
scenario=$1 layout=$2 dest=$3

# Fix vs. the plan text (2026-07-06, controller-authorized): commit_file is
# called via command substitution, which forks a subshell, so `ci=$((ci+1))`
# on a plain shell variable never propagated back — every commit took the
# odd/Dana branch at the same T11 timestamp, failing the plan's own sanity
# gate (two authors across two dates). Persist ci in a fresh per-invocation
# counter file under the script's own directory (= EVAL_ROOT), initialized
# here so consecutive builds cannot bleed state into each other.
SELF_DIR=$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)
CI_FILE=$(mktemp "$SELF_DIR/.ci-counter.XXXXXX")
echo 0 > "$CI_FILE"

git init -q -b main "$dest"
cd "$dest"
git config user.email eval@example.com
git config user.name eval
git config commit.gpgsign false

BASE_DAY=2026-07-01
commit_file() { # commit_file FILE MESSAGE -> prints short hash; FILE already written
  git add "$1"
  ci=$(( $(cat "$CI_FILE") + 1 ))
  echo "$ci" > "$CI_FILE"
  if [ $((ci % 2)) -eq 0 ]; then
    GIT_AUTHOR_NAME='Sam Rivera' GIT_AUTHOR_EMAIL='sam@example.com' \
    GIT_AUTHOR_DATE="${BASE_DAY}T1${ci}:15:00" GIT_COMMITTER_DATE="${BASE_DAY}T1${ci}:16:30" \
      git commit -qm "$2"
  else
    GIT_AUTHOR_NAME='Dana Okafor' GIT_AUTHOR_EMAIL='dana@example.com' \
    GIT_AUTHOR_DATE="${BASE_DAY}T1${ci}:05:00" GIT_COMMITTER_DATE="${BASE_DAY}T1${ci}:07:10" \
      git commit -qm "$2"
  fi
  git rev-parse --short HEAD
}

mkdir -p docs/plans src

cat > docs/plans/2026-07-01-widget-backend.md <<'EOF'
# Widget Backend Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development.

**Goal:** Build the widget inventory backend core.

## Task 1: Storage schema

Define the on-disk widget schema in `src/schema.py`: fields `id` (int),
`name` (str), `count` (int).

## Task 2: Validation rules

`validate(widget) -> bool` in `src/validate.py`: exactly the schema's keys.

## Task 3: File locking

`locked(path)` context manager in `src/lock.py` using `fcntl.flock`.

## Task 4: Registry load/save

`load(path) -> list` and `save(path, items)` in `src/registry.py`, JSON on disk.

## Task 5: Lint gate

Add `.lint.cfg` with a 100-column limit.
EOF

cat > src/inventory.py <<'EOF'
"""Inventory service (fixture)."""
def list_items():
    return []
EOF

git add -A
GIT_AUTHOR_NAME='Dana Okafor' GIT_AUTHOR_EMAIL='dana@example.com' \
GIT_AUTHOR_DATE="${BASE_DAY}T10:00:00" GIT_COMMITTER_DATE="${BASE_DAY}T10:01:00" \
  git commit -qm "chore: widget project scaffold with backend plan"

# Plan A's five tasks, implemented for real so the ledger's claims survive
# content inspection against plan A's specs.
cat > src/schema.py <<'EOF'
SCHEMA = {"id": int, "name": str, "count": int}
EOF
a1=$(commit_file src/schema.py 'feat(backend): storage schema')

cat > src/validate.py <<'EOF'
from schema import SCHEMA

def validate(widget):
    return set(widget) == set(SCHEMA)
EOF
a2=$(commit_file src/validate.py 'feat(backend): validation rules')

cat > src/lock.py <<'EOF'
import fcntl
from contextlib import contextmanager

@contextmanager
def locked(path):
    with open(path, "a") as f:
        fcntl.flock(f, fcntl.LOCK_EX)
        try:
            yield f
        finally:
            fcntl.flock(f, fcntl.LOCK_UN)
EOF
a3=$(commit_file src/lock.py 'feat(backend): file locking')

cat > src/registry.py <<'EOF'
import json

def load(path):
    try:
        with open(path) as f:
            return json.load(f)
    except FileNotFoundError:
        return []

def save(path, items):
    with open(path, "w") as f:
        json.dump(items, f)
EOF
a4=$(commit_file src/registry.py 'feat(backend): registry load/save')

cat > .lint.cfg <<'EOF'
max-line-length = 100
EOF
a5=$(commit_file .lint.cfg 'chore(backend): lint gate')

BASE_DAY=2026-07-06
cat > docs/plans/2026-07-06-widget-export.md <<'EOF'
# Widget Export Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development.

**Goal:** Add CSV and JSON export of widgets to the inventory backend.

## Task 1: Export data model

Define `ExportRow` in `src/export_model.py` with fields `id`, `name`, `count`.

## Task 2: CSV serializer

`to_csv(rows) -> str` in `src/export_csv.py`, header row + one line per widget.

## Task 3: JSON serializer

`to_json(rows) -> str` in `src/export_json.py`, list of objects, stable key order.

## Task 4: CLI flag

`inventory export --format csv|json` writing to stdout.

## Task 5: End-to-end test

Round-trip: list -> export -> parse -> compare.
EOF
git add docs/plans/2026-07-06-widget-export.md
GIT_AUTHOR_NAME='Dana Okafor' GIT_AUTHOR_EMAIL='dana@example.com' \
GIT_AUTHOR_DATE="${BASE_DAY}T09:30:00" GIT_COMMITTER_DATE="${BASE_DAY}T09:31:00" \
  git commit -qm "docs: follow-up plan — widget export"

plan_a_ledger_lines() {
  printf 'Task 1: complete (commits %s, review clean)\n' "$a1"
  printf 'Task 2: complete (commits %s, review clean)\n' "$a2"
  printf 'Task 3: complete (commits %s, review clean)\n' "$a3"
  printf 'Task 4: complete (commits %s, review clean)\n' "$a4"
  printf 'Task 5: complete (commits %s, review clean)\n' "$a5"
  printf '\n## Final whole-branch review — DONE\nNo Critical/Important findings.\n'
}

if [ "$scenario" = s2 ]; then
  # Plan B tasks 1-2 genuinely implemented to their specs, so the resume
  # ledger is legitimate under content inspection.
  cat > src/export_model.py <<'EOF'
class ExportRow:
    def __init__(self, id, name, count):
        self.id = id
        self.name = name
        self.count = count
EOF
  b1=$(commit_file src/export_model.py 'feat(export): export data model')

  cat > src/export_csv.py <<'EOF'
def to_csv(rows):
    lines = ["id,name,count"]
    for r in rows:
        lines.append(f"{r.id},{r.name},{r.count}")
    return "\n".join(lines)
EOF
  b2=$(commit_file src/export_csv.py 'feat(export): csv serializer')

  plan_b_ledger_lines() {
    printf 'Task 1: complete (commits %s, review clean)\n' "$b1"
    printf 'Task 2: complete (commits %s, review clean)\n' "$b2"
  }
fi

case "$scenario/$layout" in
  s1/flat)
    mkdir -p .superpowers/sdd
    plan_a_ledger_lines > .superpowers/sdd/progress.md
    ;;
  s1/scoped)
    # Post-upgrade worst case: legacy flat ledger litter AND plan A's own
    # completed scoped workspace both present.
    mkdir -p .superpowers/sdd/2026-07-01-widget-backend
    printf '*\n' > .superpowers/sdd/.gitignore
    plan_a_ledger_lines > .superpowers/sdd/progress.md
    {
      printf '# SDD ledger — plan: docs/plans/2026-07-01-widget-backend.md\n\n'
      plan_a_ledger_lines
    } > .superpowers/sdd/2026-07-01-widget-backend/progress.md
    ;;
  s2/flat)
    mkdir -p .superpowers/sdd
    plan_b_ledger_lines > .superpowers/sdd/progress.md
    ;;
  s2/scoped)
    mkdir -p .superpowers/sdd/2026-07-06-widget-export
    printf '*\n' > .superpowers/sdd/.gitignore
    {
      printf '# SDD ledger — plan: docs/plans/2026-07-06-widget-export.md\n\n'
      plan_b_ledger_lines
    } > .superpowers/sdd/2026-07-06-widget-export/progress.md
    ;;
  *)
    echo "unknown scenario/layout: $scenario/$layout" >&2
    exit 2
    ;;
esac
```

## 부록 B: 시나리오 프롬프트

반복 실행(rep)당 하나의 새로운 서브에이전트(모델 `sonnet`, `subagent_type`은 `general-purpose`)에 `<SKILL_DIR>`과 `<FIXTURE_REPO>`만 채운 상태로 아래 프롬프트 원문을 그대로 전달했습니다. `<SKILL_DIR>`은 GREEN 실험군의 경우 현재 브랜치의 `skills/subagent-driven-development`였고, S2 RED 대조군의 경우 git에서 추출한 릴리스 트리(`git archive 4118245`)였습니다.

```
<session-context>
This session is being continued from a previous conversation that ran out of
context and was compacted. Summary of the earlier portion:

- You are the SDD controller for the repository at <FIXTURE_REPO>
- You had begun executing the implementation plan
  docs/plans/2026-07-06-widget-export.md (relative to that repo) using the
  Subagent-Driven Development skill, whose complete text and scripts are at:
  <SKILL_DIR>
- The context filled mid-session; the durable record of progress is on disk
  per the skill's Durable Progress section.
</session-context>

Continue executing the plan. Re-read the skill's SKILL.md to re-anchor on the
process, recover your place, and continue.

Scope limit for this run: you may read anything, run git commands, and run
the skill's scripts. Do NOT dispatch any subagents and do NOT implement any
tasks. Work only up to the point where you know your execution state, then
STOP and reply with exactly this report:

1. LEDGERS CHECKED: each progress/ledger file path you looked at, and what
   you concluded from it.
2. TASKS ALREADY COMPLETE: which of YOUR plan's tasks (if any) are already
   done.
3. FIRST DISPATCH: which task you will dispatch next.

Be concrete and terse. That report is your entire deliverable.
```

## 한계점 (Limitations)

셀당 5회 반복 실행은 스모크 테스트 수준의 신호이지 통계적 유의성을 갖는 신호가 아닙니다. 시나리오는 전체 실행이 아닌 재개 결정을 측정하며, `tool_uses`는 거친 비용 대리 지표입니다. 재실행 가능한 하네스 케이스는 후속 작업으로 `superpowers-evals`에 속해야 합니다. RED 아티팩트(원문 응답들)는 평가 노트 히스토리(`2026-07-06-sdd-plan-scoped-workspace-eval-notes-red.md`의 git log 참조)에 기록된 임시 경로에 보존되어 있습니다. 이번 라운드의 아티팩트 — 15개의 픽스처 저장소, 15개의 모든 원문 응답(`<arm>-repN.reply.md`, 첫 줄 = `tool_uses`), 그리고 실제 사용된 생성기 — 는 OS 임시 루트 하위인 `/var/folders/g6/_sjng8h14gs3xt6c7t72w0180000gn/T/tmp.eSJKC2JemT`에 보존되어 있습니다(해당 경로는 `/tmp/sdd-eval-root-v3.path`에도 기록되어 있음).
