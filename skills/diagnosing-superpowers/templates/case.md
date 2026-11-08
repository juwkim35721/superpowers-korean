# Case: <session-id>

Workspace: ~/.superpowers/diagnosing-superpowers/<session-id>/
Created: <ISO timestamp>

## 문제 정의 (Problem statement — 인간 파트너(human partner)와 합의됨)

<한 단락으로 작성. 대상 세션(들), 알려진 경우 턴 범위, 기대했던 동작, 실제로 일어난 일, 그리고 핵심이 되는 관측 지표(실제 소요 시간, 토큰, 반복된 행동, 특정 예상치 못한 행동 등)를 명시합니다.>

목표가 Superpowers 버그 리포트인가: yes | no

## 세션 목록 (Sessions)

| 역할 (Role) | 세션 ID (Session id) | 절대 경로 (Absolute path) | 줄 수 (Lines) | 바이트 (Bytes) | 가장 긴 줄 (Longest line, bytes) | 첫 번째 프롬프트 (First prompt, 첫 120자) | 첫 번째 타임스탬프 (First timestamp) |
|---|---|---|---|---|---|---|---|
| main | | | | | | | |
| subagent | | | | | | | |

제외된 후보(Rejected candidates): `<id — 경로 — 제외 사유>`, 또는 `"none"`.

읽는 시점에 세션이 아직 실행 중인가(Session still running at read time): yes | no (mtime `<ISO>`, lines `<N>`)

## 환경 (Environment)

- OS: `<이름 및 버전>`
- Harness: `<이름> <버전>`
- Models seen: `<모델 id — 등장 위치 (main / subagent id)>`
- Superpowers install root: `<경로>`; version `<x.y.z>`; git sha `<sha 또는 "not a checkout">`
- 세션 중 읽거나 주입된 스킬 파일 (Skill files read or injected during the session):

| 스킬 / 소스 경로 (Skill / source path) | sha1 또는 unavailable | 출처 성격 (Provenance) | 뒷받침하는 위치 (Supporting location) |
|---|---|---|---|

환경 및 스킬 관측 결과들을 과거 증거(historical evidence), 미검증 스냅샷(unverified snapshot), 현재 관측값(current observation), 또는 알 수 없음(unknown)으로 라벨링하세요. 과거 정보를 확인할 수 없다(unavailable)고 선언하기 전에 제공된 출처 메모, 아카이브, 캡처된 스킬 본문을 먼저 확인하세요. 원본 경로가 사라졌다고 해서 보존된 복사본까지 지워지는 것은 아닙니다. 현재 버전이나 수정 시간(mtime)은 과거 세션 당시의 버전을 입증하지 못하며, 캡처된 스킬 본문 하나가 설치 환경 전체를 입증하지도 않습니다.

- 설정된 기타 플러그인 / 확장 기능 / MCP 서버: `<목록, 또는 "none found">`
- 존재하는 지침 파일 (경로만 표기): `<목록>`

## 이 파일들을 읽는 모든 주체를 위한 컨텍스트 안전 규칙 (Context-safety rules for every reader of these files)

- 여기에 나열된 어떤 파일이든 읽기 전에 `references/context-safety.md`를 따르세요.
- 서브에이전트 트랜스크립트에서 `"user"`는 부모 에이전트(parent agent)입니다.

## 발견된 소스 및 레코드 의미 (Discovered sources and record meanings)

- 참조한 소스(Sources consulted): `<절대 경로, 도구, 도움말 또는 문서 소스>`
- 추출 명령어 또는 쿼리(Extraction commands or queries): `<각 소스에 사용한 범위 제한 명령어 또는 도구 쿼리>`
- 대상 세션 식별 증거(Target identity evidence): `<세션 id, 작업 디렉터리, 타임스탬프, 일치하는 내용, 뒷받침하는 레코드 위치>`
- 연관된 세션(Associated sessions): `<세션 id, 관계, 뒷받침하는 레코드 위치, 또는 "none found">`
- 인간 메시지(Human messages): `<레코드 형태 및 그 의미에 대한 증거>`
- 주입된 메시지 및 부모 디스패치(Injected messages and parent dispatches): `<레코드 형태 및 그 의미에 대한 증거>`
- 어시스턴트 메시지(Assistant messages): `<레코드 형태 및 그 의미에 대한 증거>`
- 도구 호출 및 결과(Tool calls and results): `<레코드 형태, 상호 매칭 방식, 그리고 그 의미에 대한 증거>`
- 사용량 카운터(Usage counters): `<필드, 증분 또는 누적 의미 체계, 단위, 증거, 또는 "unavailable">`
- 타이밍(Timing): `<필드, 단위, 이벤트 경계, 증거, 또는 "unavailable">`
- 기타 관련 레코드(Other relevant records): `<모델, 버전, 컴팩션 또는 기타 의미와 증거>`
- 미해결 정보(Unresolved information): `<누락되었거나, 접근 불가능하거나, 모호하거나, 존재하지 않는 정보, 또는 "none">`
