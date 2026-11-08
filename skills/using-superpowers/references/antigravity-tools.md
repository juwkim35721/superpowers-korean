# Antigravity CLI (`agy`) 도구 매핑 (Tool Mapping)

스킬들은 동작 단위로 표현합니다("서브에이전트 디스패치", "할 일(todo) 생성", "파일 읽기"). Antigravity CLI (`agy`)에서 이러한 동작들은 아래 도구들로 매핑됩니다.

| 스킬이 요청하는 동작 | Antigravity CLI 대응 도구 |
|----------------------|----------------------|
| 서브에이전트 디스패치 (`Subagent (general-purpose):` 템플릿) | 내장 `TypeName`을 지정한 `invoke_subagent` — 전체 권한 작업은 `self`, 읽기 전용 작업은 `research` |
| 작업 추적 ("할 일(todo) 생성", "완료 표시") | **태스크 아티팩트(task artifact)** — `IsArtifact: true` 및 `ArtifactType: "task"`를 지정한 `write_to_file` ([작업 추적](#task-tracking) 참조). 백그라운드 프로세스를 관리하는 `manage_task`가 **아닙니다**. |

## 작업 추적 {#task-tracking}

Antigravity에는 **전용 할 일(todo) 도구가 없습니다** (`manage_task`는 `list`/`kill`/`status`/`send_input` 등 백그라운드 프로세스를 관리하는 도구이며, 체크리스트가 *아닙니다*). 스킬에서 할 일 목록을 만들거나 작업을 추적하라고 지시할 때는 **태스크 아티팩트(task artifact)**를 유지하세요. 즉, `write_to_file`(`IsArtifact: true`, `ArtifactMetadata.ArtifactType: "task"`)로 저장한 마크다운 체크리스트를 만들고, 작업을 진행하면서 `replace_file_content` / `multi_replace_file_content`로 편집하세요.

여러 단계로 이루어진 작업을 시작할 때는 계획의 모든 단계를 나열한 태스크 아티팩트를 생성하세요. 각 단계를 완료할 때마다 아티팩트를 편집하여 완료 표시(`- [x]`)를 하세요. 계획이 변경되면 체크리스트를 업데이트하세요. 항상 최신 상태를 유지해야 합니다. 이 아티팩트는 남은 작업에 대한 단일 진실 공급원(source of truth)이므로, 대화가 길어지면 각 단계를 시작하기 전에 다시 읽으세요.
