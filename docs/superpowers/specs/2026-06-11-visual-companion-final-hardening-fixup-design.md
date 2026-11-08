# 비주얼 컴패니언(Visual Companion) 최종 보안 강화 수정(Fixup) 설계

**날짜:** 2026-06-11
**상태:** Drew 리뷰용 초안

## 목표

PR #1720 비주얼 컴패니언 보안 강화 패스를 마무리하여, 깔끔한 보안 동작, 결정론적 테스트, 그리고 오직 컴패니언 작업만 포함된 PR diff를 갖춘 상태로 브랜치가 Jesse의 리뷰를 받을 수 있도록 준비합니다.

이 문서는 기존 인증 강화 설계 위에 적용되는 수정(fixup) 작업입니다. 컴패니언을 재설계하거나 기능 범위를 확장해서는 안 됩니다.

## 배경

이전 보안 강화 패스에서는 키 기반 세션, 동일 출처(same-origin) WebSocket 검사, URL 키 제거, `/files/*` 격리, 유출 방지 헤더, IPv6 URL 포맷팅, Windows 라이프사이클 커버리지 및 PR 증거 업데이트를 추가했습니다.

최종 리뷰 패스에서 다섯 가지 남은 이슈가 발견되었습니다:

1. 루트 `GET /` 화면 선택 경로가 여전히 콘텐츠 디렉터리 외부를 가리키는 `content/` 하위의 심볼릭 링크나 하드링크를 제공할 수 있습니다.
2. 선호 포트가 사용 중일 때 폴백(fallback) 서버가 영구 저장된 `.last-token`을 재사용할 수 있어, 동일한 베어러(bearer) 키를 공유하는 두 개의 활성 동일 프로젝트 컴패니언 서버가 생성될 수 있습니다.
3. 강력한 소유권 증명을 사용할 수 없을 때 `stop-server.sh`가 관련 없는 `node server.cjs` 프로세스에 시그널을 보낼 수 있습니다.
4. 일부 테스트가 잘못된 폴백 프로세스를 상대로 통과하거나, 실패 시 백그라운드 프로세스를 누수하거나, Windows 계열 호스트에서 심볼릭 링크 지원을 단정합니다.
5. 별도로 처리되었던 이전 `evals` 서브모듈 버전 업데이트가 브랜치에 포함되어 있어 현재 PR에 충돌(conflict)이 발생해 있습니다.

## 비목표 (Non-Goals)

- 이번 패스에서는 HTTPS 터널이나 `wss://` 출처 시맨틱을 추가하지 않습니다.
- 옵트아웃(opt-out), 자유 텍스트, 대비 보조 도구(contrast-helper) 컴패니언 기능을 구현하지 않습니다.
- Alpine, Three.js 또는 기타 어떠한 JavaScript 라이브러리도 번들링(vendor)하지 않습니다.
- 에이전트가 악의적으로 작성한 화면 HTML을 샌드박싱하려고 시도하지 않습니다.
- Drew가 해당 트레이드오프를 명시적으로 승인하지 않는 한, 오래된(stale) stop-server PID 파일에 대한 하위 호환성을 추가하지 않습니다.

## 계승된 보안 불변 조건 (Inherited Security Invariants)

이번 수정 작업은 이미 설계되고 구현된 인증 강화 사항들을 그대로 보존합니다:

- `.last-token`과 `state/server-info`는 민감한 소유자 전용(owner-only) 상태로 유지됩니다.
- 폴백 토큰은 시작 JSON과 `state/server-info`에는 나타날 수 있지만, `.last-token`에 기록되어서는 안 됩니다.
- 쿠키는 포트 이름이 지정되고, `HttpOnly`, `SameSite=Strict`가 적용되며, `/` 범위로 유지됩니다.
- WebSocket 업그레이드는 여전히 유효한 키 또는 쿠키를 필요로 합니다.
- 브라우저가 `Origin` 헤더를 제공할 때 WebSocket `Origin` 검사가 계속 강제됩니다.
- `Origin`이 없는 직접 연결 클라이언트는 세션 키를 보유한 경우에만 계속 허용됩니다.
- 생성된 동일 출처 화면 JavaScript와 향후 동일 출처 번들 라이브러리는 신뢰할 수 있는 것으로 간주합니다. 악의적인 화면 HTML 샌드박싱은 계속 보류 상태로 둡니다.

## 설계

### 1. 최신 `dev` 브랜치로 리베이스

구현 작업을 시작하기 전에 `brainstorming-companion` 브랜치를 최신 `origin/dev` 위로 리베이스합니다. `evals` 서브모듈 충돌은 `dev` 버전을 선택하여 해결합니다.

리베이스 이후:

- PR diff에 `evals`가 나타나지 않아야 합니다.
- PR #1720은 다른 곳에서 실행된 평가(eval) 증거를 계속 언급할 수 있지만, 정확한 외부 증거(평가 저장소 커밋, 시나리오 경로, 명령어, 결과 아티팩트 경로 또는 ID, RED/GREEN 결과)를 포함해야 합니다.
- PR 본문은 `evals` 서브모듈 업데이트가 이 PR의 일부인 것처럼 암시해서는 안 됩니다.
- 서브모듈 업데이트가 포함되어 있음을 암시하는 이전 PR 본문 텍스트나 코멘트는 최종 PR 본문 증거로 대체되어야 합니다.

### 2. 루트 화면 격리 (Root Screen Containment)

루트 화면 라우트는 `/files/*`와 동일한 격리 경계를 사용해야 합니다.

`getNewestScreen()`은 콘텐츠 디렉터리 내부의 일반 파일 가드를 통과하지 못하는 모든 `.html` 후보를 무시해야 합니다. 이 가드는 실제 경로(real path)를 확인하고 제공되는 파일이 `CONTENT_DIR` 내부에 있는지 보장해야 합니다. 또한 플랫폼이 링크 카운트를 보고할 때 링크 카운트가 정확히 1이 아닌 파일을 거부함으로써 기존의 하드링크 보호 기능을 보존해야 합니다.

예상 동작:

- `content/` 외부를 가리키는 `content/` 하위의 심볼릭 링크는 무시됩니다.
- `fs.linkSync`가 성공하고 `lstat.nlink > 1`일 때, `state/server-info`를 가리키는 `content/` 하위의 하드링크는 무시됩니다.
- 안전한 화면 파일이 남아 있지 않으면 대기 페이지(waiting page)가 제공됩니다.
- 기존의 `/files/*` 격리 동작은 변경되지 않습니다: 빈 이름, 숨김 파일(dotfiles), 심볼릭 링크, 하드링크, 디렉터리는 여전히 404를 반환합니다.

### 3. 폴백 토큰 격리 (Fallback Token Isolation)

포트 폴백은 영구 저장된 `.last-token`에서 로드한 토큰을 재사용해서는 안 됩니다.

토큰 출처는 코드에서 명시적으로 구분되어야 합니다:

- 환경 변수 `BRAINSTORM_TOKEN`은 의도적인 운영자/테스트 오버라이드입니다. 명시적인 환경 변수 토큰이 설정된 상태에서 선호 포트가 사용 중이라면, 해당 포트를 점유한 서버가 동일한 명시적 토큰을 사용하고 있을 수 있으므로 서버는 폴백하는 대신 안전하게 실패(fail closed)해야 합니다.
- `.last-token`은 동일 포트 재연결 편의를 위해 영구 저장된 상태입니다. 선호 포트가 사용 중이어서 서버가 폴백하는 경우, 로드된 토큰을 폐기하고 폴백 프로세스를 위해 영구 저장되지 않는 새로운 토큰을 생성합니다.
- `.last-token`에서 로드되지 않고 새로 생성된 토큰은 다른 활성 프로세스가 이를 가지고 있지 않음이 알려져 있으므로 동일 프로세스 내에서 재사용할 수 있습니다.

폴백 서버는 계속해서 `.last-port`와 `.last-token`을 덮어쓰지 않아야 합니다.

### 4. Stop-Server 소유권 증명

`start-server.sh`는 시작할 때마다 고유한 서버 인스턴스 ID를 생성하여 비활성 명령줄 인자로 Node에 전달해야 합니다. 예:

```text
node server.cjs --brainstorm-server-id=<id>
```

이 ID는 인증 자격 증명이 아닙니다. 로컬 라이프사이클 스크립트를 위한 프로세스 소유권 증거일 뿐입니다. `server.cjs`는 이 인자를 무시해도 됩니다.

ID는 `^[A-Za-z0-9_-]{32,64}$`와 같이 셸/MSYS에 안전한 알파벳을 사용해야 합니다. 이를 소유자 전용 권한으로 `state/server-instance-id`에 저장합니다.

`stop-server.sh`는 상태에서 예상 ID를 읽고, 대상 프로세스 argv가 느슨한 부분 문자열이 아니라 전체 argv 토큰으로서 정확한 인자 `--brainstorm-server-id=<id>`를 포함할 때만 PID에 시그널을 보내야 합니다. 가능한 경우 `/proc/<pid>/cmdline`을 우선 사용하고, 그렇지 않으면 넓은 출력의 `ps`로 폴백합니다. 일치하는 인스턴스 ID가 있으면 `server-info`가 없거나 `lsof`를 사용할 수 없는 경우에도 충분한 증명이 됩니다. 기존의 포트-PID 대조 검사는 추가 증거로 남겨둘 수 있습니다.

소유권을 증명할 수 없는 다음 경우에는 안전하게 실패(fail closed)합니다:

- PID 파일 누락
- 서버 ID 누락 또는 형식 오류
- 대상 명령줄 확인 불가
- 대상 명령줄에 예상 ID가 포함되지 않음
- 새로운 ID가 없는 이전/오래된(stale) 세션 메타데이터

이는 관련 없는 프로세스를 종료하는 것보다 오래된 프로세스를 실행 상태로 남겨두는 것을 의도적으로 선호하는 설계입니다.

운영자가 확인할 수 있는 결과는 명확해야 합니다:

- PID 파일 누락은 `not_running` 반환
- 서버 ID 누락 또는 형식 오류는 `stale_pid` 반환
- 명령줄 확인 불가는 `stale_pid` 반환
- 잘못되었거나 없는 argv ID는 `stale_pid` 반환
- 성공적인 중지는 `stopped` 반환

`stale_pid` 및 `stopped` 결과가 나오면 향후 중지 시도가 동일한 모호한 프로세스를 계속 타겟팅하지 않도록 `server.pid`와 `server-instance-id`를 제거합니다. 영구 세션 콘텐츠는 제거하지 마세요.

### 5. 테스트 강화

테스트 통과는 검증에 사용되는 macOS와 Windows Git Bash 호스트 전반에서 결정론적이어야 합니다.

필수 변경 사항:

- 고정 포트 스위트는 서버가 폴백 포트를 보고할 경우 즉시 실패(fail fast)하거나, 보고된 시작 포트에서 모든 클라이언트를 구동해야 합니다.
- `stop-server.test.sh`는 백그라운드 프로세스가 시작되기 전에 최상위 정리 트랩(cleanup trap)을 설정해야 합니다.
- 심볼릭 링크 관련 단언문(assertion)은 심볼릭 링크 기능을 먼저 프로브하고, 호스트가 사용 가능한 테스트 심볼릭 링크를 생성할 수 없는 경우에만 해당 단언문을 건너뛰어야 합니다.
- 사칭(impostor) 프로세스를 생성하는 테스트는 라이프사이클 메타데이터가 누락되었거나 불충분할 때 사칭 프로세스가 살아남는지 확인해야 합니다.
- Windows/MSYS `start-server` 테스트는 Windows 계열 감지가 여전히 `BRAINSTORM_OWNER_PID`를 지우고, 적절한 경우 자동으로 포그라운드로 전환하며, 인스턴스 ID argv를 정확하게 전달하는지 확인해야 합니다.

### 6. 문서 및 PR 일관성

Jesse의 리뷰 전에 리뷰어가 볼 수 있는 문서와 PR 메타데이터를 일치시킵니다:

- 이슈 카탈로그를 업데이트하여 처리 결과(dispositions)가 이 PR이 실제로 출시하는 내용과 일치하도록 합니다.
- 자동 열기 문서를 구현된 `--open` 동작과 일관되게 유지합니다.
- 문서화된 기본 유휴 타임아웃을 모든 곳에서 4시간으로 유지합니다.
- 리베이스 후 PR 본문을 템플릿과 대조하여 검토합니다.
- macOS, Windows, 브라우저/수동 테스트 및 외부 평가(eval) 증거를 구체적인 명령어 및 결과와 함께 PR 본문에 기록합니다.

## 테스트 전략

각 동작 변경에 대해 TDD를 사용합니다:

1. 집중 회귀 테스트를 추가하거나 강화합니다.
2. 테스트를 실행하고 예상한 이유로 실패하는지 확인합니다.
3. 가장 작은 수정 사항을 구현합니다.
4. 집중 테스트를 재실행합니다.
5. 전체 brainstorm-server 스위트를 재실행합니다.

필수 집중 회귀 테스트 항목:

| 동작 | 테스트 파일 | 집중 명령어 | 예상 RED | 예상 GREEN |
| --- | --- | --- | --- | --- |
| 루트 라우트가 심볼릭 링크 탈출 무시 | `tests/brainstorm-server/server.test.js` | `node tests/brainstorm-server/server.test.js` | 인증된 `GET /`이 외부로 링크된 콘텐츠 제공 | 응답이 대기 페이지 또는 안전한 화면 제공 |
| 루트 라우트가 지원되는 하드링크 탈출 무시 | `tests/brainstorm-server/server.test.js` | `node tests/brainstorm-server/server.test.js` | 인증된 `GET /`이 하드링크된 `server-info` 제공 | `nlink > 1`일 때 하드링크 후보 무시 |
| `/files/*` 격리 유지 | `tests/brainstorm-server/server.test.js` | `node tests/brainstorm-server/server.test.js` | 기존 격리 테스트 회귀 발생 | 빈 이름, 숨김 파일, 디렉터리, 심볼릭 링크, 하드링크 케이스가 404 유지 |
| 영구 토큰 폴백 시 토큰 교체 | `tests/brainstorm-server/lifecycle.test.js` | `node tests/brainstorm-server/lifecycle.test.js` | 폴백 URL 키가 영구 저장된 선호 포트 키와 동일함 | 폴백 URL 키가 다르며 `.last-token`에 기록되지 않음 |
| 명시적 토큰 폴백 시 안전하게 실패 | `tests/brainstorm-server/lifecycle.test.js` | `node tests/brainstorm-server/lifecycle.test.js` | `BRAINSTORM_TOKEN`이 설정된 상태에서 서버가 폴백함 | 프로세스가 0이 아닌 코드로 종료되고 폴백을 시작하지 않음 |
| 폴백 키로 원래 서버에 인증 불가 | `tests/brainstorm-server/lifecycle.test.js` | `node tests/brainstorm-server/lifecycle.test.js` | 폴백 키가 원래 포트로부터 200 응답을 받음 | 원래 포트가 폴백 키를 거부함 |
| 올바른 인스턴스 ID가 중지 허용 | `tests/brainstorm-server/stop-server.test.sh` | `bash tests/brainstorm-server/stop-server.test.sh` | 실제 start-server로 실행된 서버가 살아남음 | stop이 `stopped`를 반환하고 프로세스가 종료됨 |
| 잘못되거나 누락되거나 형식 오류가 있거나 오래된 ID는 안전함 | `tests/brainstorm-server/stop-server.test.sh` | `bash tests/brainstorm-server/stop-server.test.sh` | 사칭 프로세스에 시그널이 전달됨 | stop이 `stale_pid`를 반환하고 사칭 프로세스가 살아남음 |
| 고정 포트 스위트가 폴백을 통해 통과할 수 없음 | `tests/brainstorm-server/server.test.js`, `tests/brainstorm-server/auth.test.js` | 각 `node` 명령어 | 테스트가 조용히 폴백 포트와 통신함 | 테스트가 명확하게 실패하거나 보고된 포트를 의도적으로 사용함 |
| 실패 시 셸 정리 트랩 실행 | `tests/brainstorm-server/stop-server.test.sh` | `bash tests/brainstorm-server/stop-server.test.sh` | 실패 시 자식 프로세스가 남음 | 트랩이 백그라운드 자식 프로세스를 수거함 |
| Windows/MSYS 시작 동작이 라이프사이클 불변 조건 유지 | `tests/brainstorm-server/start-server.test.sh`, `tests/brainstorm-server/windows-lifecycle.test.sh` | macOS 및 `ballmer`에서의 `bash` 테스트 명령어 | 소유자 PID 또는 argv 처리에서 회귀 발생 | 소유자 PID가 지워지고, 포그라운드 감지가 유지되며, ID argv가 존재함 |

각 RED/GREEN 사이클은 PR 본문을 위한 짧은 증거 노트를 남겨야 합니다: 집중 명령어, 수정 전 실패한 단언문, 수정 후 통과한 단언문, 그리고 증거가 수집된 환경(macOS 또는 Windows).

## 검증 (Verification)

수정 작업이 완료되었다고 선언하기 전에 다음을 실행하세요:

- `git fetch origin dev && git rebase origin/dev`
- `git diff --quiet origin/dev...HEAD -- evals`
- `gh pr view 1720 --json mergeStateStatus,statusCheckRollup,headRefOid`
- `cd tests/brainstorm-server && npm test`
- TDD 중 사용된 관련 집중 테스트 명령어들
- `git diff --check`
- 수정된 JavaScript 파일에 대한 Node 구문 검사
- 수정된 셸 파일에 대한 셸 린트
- `ballmer`에서의 Windows 검증: 실행 가능한 전체 brainstorm-server 스위트 및 독립 실행형 Windows 라이프사이클 프로브

수동/브라우저 테스트는 자동화 패스가 모두 통과(green)한 후에만 진행합니다.

## 인수 기준 (Acceptance Criteria)

- PR #1720이 최신 `dev` 위로 깔끔하게 리베이스됩니다.
- PR diff에 `evals`가 존재하지 않습니다.
- 루트 화면 제공 경로가 심볼릭 링크나 지원되는 하드링크 탈출을 통해 `content/` 외부를 읽을 수 없습니다.
- `/files/*` 격리 보호 기능이 변경 없이 유지됩니다.
- 어떠한 폴백 서버도 점유된 선호 포트 서버와 공유될 수 있는 토큰으로 실행되지 않습니다.
- 소유권 증명이 누락되었거나 모호할 때 `stop-server.sh`가 관련 없는 프로세스에 시그널을 보내지 않습니다.
- `server-info`나 `lsof`를 사용할 수 없을 때도 `stop-server.sh`가 일치하는 인스턴스 ID를 가진 정상 서버를 중지할 수 있습니다.
- 각 회귀 항목에 대해 집중 RED/GREEN 증거가 기록됩니다.
- macOS 및 Windows 검증 증거가 PR 본문에 기록됩니다.
- PR 본문이 브랜치에 포함된 내용과 외부에서 수집된 증거를 정확하게 설명합니다.
