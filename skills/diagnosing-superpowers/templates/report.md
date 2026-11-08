# 세션 진단 (Session diagnosis): <session-id>

Report path: ~/.superpowers/diagnosing-superpowers/<session-id>/report.md
Written: <ISO timestamp>

## 1. 문제 정의 (Problem statement, 필수)

<케이스 파일에서 복사한 내용.>

## 2. 트리아지 판정 (Triage verdict, 필수)

<보고된 문제 전후로 실제로 무슨 일이 일어났는지 증거가 보여주는 내용. 산문 형식으로 작성하되 모든 주장 뒤에 `path:line`을 붙이세요. 신뢰도(high / medium / low)와 무엇이 있으면 신뢰도를 높일 수 있는지 명시하세요. Superpowers가 무엇을 해야 하는지에 대한 언급은 하지 마세요.>

## 3. 환경 (Environment, 필수)

- OS:
- Harness and version:
- Models seen:
- Superpowers install root / version / git sha:
- Skill files read or injected (케이스 파일의 sha1 표):
- Other plugins, extensions, MCP servers:
- Instruction files present (경로만 표기):

모든 환경 필드와 스킬 관측 결과를 과거 증거(historical evidence), 미검증 스냅샷(unverified snapshot), 현재 관측값(current observation), 또는 알 수 없음(unknown)으로 라벨링하고, 이를 뒷받침하는 증거 위치를 기록하세요.

## 4. 검사한 세션 (Sessions examined, 필수)

| 역할 (Role) | 세션 ID (Session id) | 절대 경로 (Absolute path) | 줄 수 (Lines) | 바이트 (Bytes) |
|---|---|---|---|---|

제외된 후보(Rejected candidates): `<id — 경로 — 제외 사유>`, 또는 `"none"`.

## 5. 타임라인 (Timeline, 필수)

인간이 직접 입력한 프롬프트당 한 행씩 작성합니다. Events 열에는 호출된 스킬, 디스패치된 서브에이전트, 컴팩션, 에러, 재개(resumes), 중단(aborts)을 나열합니다.

| Turn | Line | Time | Request (한 줄 요약) | Events |
|---|---|---|---|---|

## 6. 분석 결과 (Findings, 필수 — 분석 차원당 하나의 하위 섹션)

각 분석 결과 형식:
```
- finding: <한 문장>
  evidence: <path:line> — "<짧은 인용문>"
  turns: <첫 턴>–<마지막 턴>
  confidence: high | medium | low
```
보고할 내용이 없는 차원은 `none found — checked: <검사한 내용>`이라고 적습니다.

### 6.1 스킬 타임라인 (Skill timeline)
### 6.2 계획 준수 (Plan adherence)
### 6.3 반복 작업 (Repeated work)
### 6.4 차질 및 실패 (Stumbles)
### 6.5 품질 증거 (Quality evidence)
### 6.6 요청 충돌 (Request conflicts)
### 6.7 비용 및 시간 (Cost and time)
### 6.8 사용된 기타 플러그인 및 스킬 (Other plugins and skills used)

## 7. Superpowers 관여도 (Superpowers involvement, 필수)

not indicated | possible | likely

증거 줄(Evidence lines): `<path:line 목록>`. 이 섹션은 관여 여부(involvement)만 기술합니다. 결함을 규정하거나 변경 사항을 제안하지 않습니다.

## 8. 분석 범위 참고사항 (Coverage notes, 필수)

- 읽지 않은 부분(Not read): `<범위, 파일, 그리고 그 이유>`
- 사용할 수 없었던 하네스 기능(Harness features unavailable): `<목록 또는 none>`
- 읽는 시점에 세션이 진행 중이었는가(Session was in progress at read time): yes/no
- 인간 파트너(human partner)가 재확인해야 할 사항(For your human partner to double-check): `<목록 또는 none>`

## 9. 유사 세션 (Similar sessions — 요청된 경우에만 작성)

| Session id | Path | Date | Harness | Matched | Did not match |
|---|---|---|---|---|---|
