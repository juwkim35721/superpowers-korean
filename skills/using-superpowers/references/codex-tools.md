## 서브에이전트 디스패치에는 멀티 에이전트 지원이 필요합니다 (Subagent dispatch requires multi-agent support)

Codex 설정 파일(`~/.codex/config.toml`)에 다음을 추가하세요:

```toml
[features]
multi_agent = true
```

이렇게 하면 `dispatching-parallel-agents` 및 `subagent-driven-development`와 같은 스킬이 사용하는 멀티 에이전트 도구가 활성화됩니다. 제공되는 구체적인 도구는 모델 프리셋이 선택하는 멀티 에이전트 버전에 따라 달라집니다(현재 프리셋은 V2를 실행하며, 구버전 프리셋은 V1을 실행합니다). 본 문서의 표를 포함한 어떤 표와 실제 도구 목록이 서로 다를 때는 항상 현재 세션의 실제 도구 목록을 신뢰하세요.

- **생성(Spawning):** `spawn_agent {fork_turns: "none"}`을 사용하여 자식 에이전트에게 깨끗한 컨텍스트를 제공하세요. 기본값인 `"all"`은 전체 대화 트랜스크립트를 자식 에이전트에 복사합니다. Codex 0.145 이상에서는 `~/.codex/agents/` 아래의 역할(role) 파일들이 `agent_type`을 통해 격리된 포크(isolated forks)에 연결됩니다. 전체 히스토리 포크(full-history forks)에서도 `model` 및 `reasoning_effort` 재정의(override)는 허용됩니다(여기서는 `agent_type`만 거부됨). 격리된 포크가 SDD(서브에이전트 기반 개발)의 기본값인 이유는 재정의를 위해서가 아니라 컨텍스트 위생(context hygiene)을 유지하기 위해서입니다.
- **수정 라운드(Fix rounds):** `followup_task`로 구현자(implementer)를 재개하세요. 이 도구는 메시지를 전달하고, 턴(turn)을 트리거하며, 하네스가 메모리에서 내보낸(evicted) 자식 에이전트를 투명하게 다시 로드합니다. 한 번 생성된 에이전트에게는 다시 메시지를 보낼 수 없다는 잘못된 가정하에 새로운 구현자를 디스패치하지 마세요. V2에서는 언제나 다시 메시지를 보낼 수 있습니다.
- **수명 주기(Lifecycle):** V2에는 `close_agent`가 없습니다. 완료된 자식 에이전트는 슬롯이 필요할 때 자동으로 정리(evict)되므로 닫지 않고 두어도 아무런 비용이 들지 않습니다. `close_agent`는 V1 세션에만 존재합니다. V1에서는 리뷰어의 리뷰가 반환되면 리뷰어를 닫고, 각 구현자는 해당 태스크의 리뷰가 통과된 후에 닫으세요.
- **모델 이름(Model names):** 현재 세션의 생성 허용 목록(spawn allowlist)과 대조해 보지 않고 스킬, 표 또는 이전 세션에서 모델 이름을 그대로 복사하여 `spawn_agent`에 넣지 마세요. V2는 V2 호환 프리셋만 허용하며 나머지 이름에 대해서는 즉시 에러(hard-error)를 발생시킵니다.

## 자식 에이전트 대기하기 (Waiting on children)

`wait_agent`는 폴링(poll)이 아니라 이벤트 구독(event subscription)입니다. 긴 대기 시간을 설정하더라도 자식 에이전트가 메일박스 활동을 생성하는 즉시 짧은 대기 시간과 동일한 지연 시간(latency)으로 깨어납니다. 짧은 타임아웃으로 반복 폴링하는 것은 아무런 이득이 없으며 폴링할 때마다 도구 호출 비용과 컨텍스트 재과금(context rebill)을 발생시킵니다. 실제 측정된 세션들에서 전체 대기 호출의 약 3분의 2가 타임아웃으로 끝난 짧은 폴링이었습니다.

- 로컬에서 처리할 작업이 남아 있는 동안에는 아예 대기하지 마세요. 완료된 자식 에이전트의 최종 응답은 메일박스로 푸시되어 다음 턴에 자동으로 도착합니다.
- 자식 에이전트들의 작업이 진행 중이고 컨트롤러가 실제로 유휴(idle) 상태일 때는 정해진 구간(bounded stretches) 단위로 대기하세요. 즉, `wait_agent`의 `timeout_ms`를 300000~600000(5~10분)으로 설정하세요. 각 대기 구간이 끝난 후(깨어났든 타임아웃되었든), 상태 줄을 한 줄 출력하고 `list_agents`를 실행한 뒤 보고 없이 종료된 자식 에이전트가 있는지 확인하여 후속 조치를 취하세요. 5분 미만의 짧은 폴링을 연속해서 쌓지 마세요. 이벤트 구독 방식이므로 긴 대기 구간도 짧은 대기 구간만큼 빠르게 깨어납니다.
- 완료 메일은 유휴 상태인 컨트롤러를 스스로 깨울 수 없습니다(턴을 트리거하지 않고 배달만 됨). 그 유휴 시간 창을 커버하는 것이 `wait_agent`의 유일한 역할입니다. 아무런 활동 없이 타임아웃된 대기 구간은 다음 대기 시간을 줄이라는 신호가 아니라 상태를 점검하고 조율(reconcile)하라는 신호입니다.

## 생성 시 모델 라우팅 (Model routing on spawns)

자신이 팬아웃(fan-out)을 실행하는 생성된 자식 에이전트인 경우를 포함하여, 발행하는 모든 `spawn_agent` 호출에서는 실행 중인 스킬의 모델 선택(Model Selection) 규칙에 따라 `model`과 `reasoning_effort`를 **둘 다** 명시적으로 설정해야 합니다. `model`만 단독으로 설정하는 것은 함정입니다. 자식 에이전트의 추론 강도(effort)가 부모의 설정이 아니라 해당 모델의 기본값으로 조용히 초기화되어 버립니다.

실수로 설정이 누락된 생성 호출이 발생하더라도 세션에서 가장 비싼 모델을 그대로 상속받는 대신 의도한 등급(tier)으로 라우팅되도록, 인간 파트너(human partner)에게 `~/.codex/config.toml`에 머신 레벨의 안전장치(backstop)를 추가해 달라고 요청하세요:

```toml
[agents]
default_subagent_model = "<a mid-tier model from your spawn allowlist>"
default_subagent_reasoning_effort = "medium"
```

## 환경 감지 (Environment Detection)

워크트리(worktree)를 생성하거나 브랜치를 마무리하는 스킬은 작업을 진행하기 전에 읽기 전용 git 명령어로 현재 환경을 먼저 감지해야 합니다:

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

- `GIT_DIR != GIT_COMMON` → 이미 연결된 워크트리(linked worktree) 내부에 있음 (워크트리 생성 건너뛰기)
- `BRANCH`가 비어 있음 → 분리된 HEAD(detached HEAD) 상태 (샌드박스에서 브랜치 생성/푸시/PR 불가)

각 스킬이 이러한 신호를 어떻게 활용하는지는 `using-git-worktrees`의 Step 0 및 `finishing-a-development-branch`의 Step 1을 참조하세요.

## Codex 앱에서 작업 마무리하기 (Codex App Finishing)

샌드박스가 브랜치 생성이나 푸시 작업을 차단하는 경우(외부에서 관리되는 워크트리 내의 분리된 HEAD 상태 등), 에이전트는 모든 작업을 커밋한 뒤 사용자에게 앱의 기본 컨트롤(native controls)을 사용하도록 안내합니다:

- **"Create branch"** — 브랜치 이름을 지정한 후 앱 UI를 통해 커밋/푸시/PR 진행
- **"Hand off to local"** — 사용자의 로컬 체크아웃 환경으로 작업 내용 이전

이 경우에도 에이전트는 테스트를 실행하고, 파일을 스테이징(stage)하며, 사용자가 복사해서 쓸 수 있도록 추천 브랜치 이름, 커밋 메시지, PR 설명을 출력해 줄 수 있습니다.
