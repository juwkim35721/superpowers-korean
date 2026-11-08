# Superpowers 세션 진단 번들 (Superpowers session diagnosis bundle)

Session: <session-id>
Harness: <name> <version> (<provenance label>)    Superpowers: <version> (<sha or "not a checkout">; <provenance label>)
Redaction level: skeleton | evidence | full
Built: <ISO timestamp>

헤더의 버전 필드들은 과거 증거(historical evidence), 미검증 스냅샷(unverified snapshot), 현재 관측값(current observation), 또는 알 수 없음(unknown)으로 출처 성격(provenance)을 명시하세요. `environment.json`에도 모든 환경 필드와 그 뒷받침 위치에 대해 동일한 출처 구분이 기록됩니다.

## 이 번들의 정체 (What this is)

Superpowers가 설치된 상태에서 문제가 발생했던 코딩 에이전트 세션의 민감정보 제거(scrubbed) 기록입니다. 이 번들을 통해 당시 현장에 없었던 에이전트나 사람이 Superpowers가 문제에 영향을 미쳤는지, 만약 그렇다면 무엇을 변경해야 하는지 판단할 수 있습니다. 내부 보고서는 `path:line` 증거와 함께 실제로 무슨 일이 일어났는지를 기술합니다. 설계상 이 번들에는 Superpowers 자체에 대한 진단이나 수정안 제안이 포함되어 있지 않으며, 그것은 읽는 사람의 몫입니다.

## 파일 구성 (Files)

- `report.md` — 진단 보고서 (문제 정의, 판정 요약, 환경, 검사한 세션, 타임라인, 분석 결과, 관여도, 분석 범위 참고사항).
- `case.md` — 분석가 에이전트들이 기준으로 삼아 작업한 케이스 파일.
- `environment.json` — 환경 섹션의 머신 판독 가능(machine-readable) 복사본.
- `timeline.md` — 턴별 타임라인.
- `findings/<dimension>.md` — 각 분석 차원별 원본 분석가 보고 결과.
- `transcripts/<session-id>.md` — 검사한 각 세션의 턴별 요약 렌더링 (원본 JSONL은 절대 포함하지 않음). 마스킹 수준(Redaction level)에 따른 도구 결과 본문 처리:

  | 수준 (Level) | 도구 결과 본문 (Tool-result bodies) |
  |---|---|
  | skeleton | 의도적으로 제한됨; `[tool result: <tool>, <bytes> bytes, exit <code>]`로 대체됨 |
  | evidence | 분석 결과를 뒷받침하는 데 필요한 명령어와 결과를 포함하여, 인용된 이벤트에 대해서만 본문 유지 |
  | full | 모두 유지 |
- `scrub-log.md` — 사용된 모든 플레이스홀더와 그 카테고리 (원본 값은 절대 포함하지 않음).

## 읽는 방법 (How to read it)

먼저 `report.md`의 §1–2를 읽고, 이어서 §7(관여도)과 거기서 인용하는 증거 줄 번호들을 확인한 뒤, `transcripts/`에서 해당하는 턴들을 살펴보세요. `path:line` 참조는 보고자의 머신에 있는 원본 파일을 가리키며, 요약된 트랜스크립트 내에도 동일한 줄 번호가 `[L<n>]` 마커로 보존되어 있습니다.

## 민감정보 마스킹 (Redaction)

플레이스홀더는 `<EMAIL-1>`, `<PERSON-2>`, `<SECRET-3>`, `<HOST-4>`, `<REPO-5>`, `<ORG-6>`, `<PROPRIETARY-7>`과 같은 형태를 띠며, 홈 경로는 `~/…`로 다시 작성됩니다. 이 번들 내에서 동일한 플레이스홀더는 항상 동일한 원본 값을 가리킵니다.

## 생성자 지침 (Producer instructions)

완성된 번들에서는 이 지침 섹션을 실제 결과로 대체합니다.

마스킹(scrubbing)을 마친 후, 오직 이 번들만을 사용하여 외부로 내보낼 모든 주요 분석 결과(material exported finding)를 검증하세요: 인용된 위치를 번들에 포함된 트랜스크립트/소스 마커에서 찾고, 인용된 명령어/결과 또는 인용문을 읽은 뒤 그것이 해당 주장을 실제로 뒷받침하는지 확인하세요. 단순히 파일 경로와 줄 번호가 존재한다는 것만으로는 불충분합니다. 마스킹 수준이나 필수적인 정보 제외로 인해 뒷받침 증거가 제거된 경우 구체적인 한계점을 기록하세요.

보고서(`report.md`), 케이스 파일(`case.md`), 환경(`environment.json`), 분석 결과(`findings/`), README 및 로컬 이슈 초안 간의 내용이 서로 일치하도록 조율(reconcile)하세요. 로그 파일 자체를 제외한 최종 파일들을 기준으로 `scrub-log.md`의 카운트를 새로 고치세요. 오래된 내보내기 문구를 제거하고, 번들 준비 단계와 아카이브 전달 단계를 명확히 구분하세요. 과거 앵커(historical anchors)에서 번들에 포함된 증거로의 매핑을 유지하세요.

독립적인 개인정보/보안 감사(privacy audit) 결과를 증거 유용성(evidence usefulness)과 분리하여 기록하세요:
- Privacy audit: `CLEAN` 또는 미해결 누락 항목(unresolved misses).
- Evidence support: 뒷받침됨(supported) 또는 제한됨(limited) (영향받은 분석 결과 및 사유 포함).

검증 후 내용이 변경되었다면 영향받은 검사를 다시 수행하세요. 아카이브 생성 승인을 받기 위해 최종 로그, 파일 목록, 그리고 위 두 가지 검증 결과를 함께 제시하세요. 검토된 파일들을 아카이브로 묶고 전달된 아카이브가 검토된 파일들과 일치하는지 확인하세요. 승인 이후에 번들 내용을 변경하지 말고, 아카이브 전달 여부는 검토된 번들 외부에 기록하세요. 스크러빙(scrubbing)은 완전무결한 개인정보 보호 인증이 아닙니다.
