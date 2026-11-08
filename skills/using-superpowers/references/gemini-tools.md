# Gemini CLI 도구 매핑 (Tool Mapping)

스킬들은 동작 단위로 표현합니다("서브에이전트 디스패치", "할 일(todo) 생성", "파일 읽기"). Gemini CLI에서 이러한 동작들은 아래 도구들로 매핑됩니다.

| 스킬이 요청하는 동작 | Gemini CLI 대응 도구 |
|----------------------|----------------------|
| 파일 읽기 | `read_file` |
| 여러 파일을 한 번에 읽기 | `read_many_files` |
| 새 파일 생성 | `write_file` |
| 파일 편집 | `replace` |
| 셸 명령어 실행 | `run_shell_command` |
| 파일 내용 검색 | `grep_search` |
| 이름으로 파일 찾기 | `glob` |
| 파일 및 하위 디렉터리 목록 보기 | `list_directory` |
| URL 가져오기 | `web_fetch` |
| 웹 검색 | `google_web_search` |
| 스킬 호출 | `activate_skill` |
| 서브에이전트 디스패치 (`Subagent (general-purpose):` 템플릿) | `agent_name: "generalist"`를 지정한 `invoke_agent` (`@generalist` 채팅 문법으로도 호출 가능 — [서브에이전트 지원](#subagent-support) 참조) |
| 다중 병렬 디스패치 | 동일한 응답 내에서 여러 번의 `invoke_agent` 호출 |
| 작업 추적 ("할 일(todo) 생성", "완료 표시") | `write_todos` (상태값: pending, in_progress, completed, cancelled, blocked) |

## 지침 파일 (Instructions file)

스킬에서 "지침 파일(your instructions file)"을 언급할 때, Gemini CLI에서 이는 **`GEMINI.md`**를 가리킵니다. Gemini CLI는 `GEMINI.md`를 계층적으로 로드합니다: `~/.gemini/GEMINI.md`의 전역 파일, 워크스페이스 디렉터리 및 그 상위 디렉터리들의 프로젝트 수준 파일, 그리고 도구가 하위 디렉터리의 파일에 접근할 때 로드되는 해당 하위 디렉터리의 `GEMINI.md` 파일입니다.

## 개인 스킬 디렉터리 (Personal skills directory)

사용자 수준 스킬은 **`~/.gemini/skills/`**에 위치하며, **`~/.agents/skills/`**를 크로스 런타임 별칭(Codex 및 Copilot CLI와 공유)으로 함께 지원합니다. 동일한 범위(scope)에 두 디렉터리가 모두 존재할 경우 `.agents/skills/`가 우선순위를 갖습니다. 각 스킬은 `SKILL.md`(`name`과 `description` 프론트매터 포함)를 담고 있는 하위 디렉터리입니다.

## 서브에이전트 지원 {#subagent-support}

Gemini CLI는 `agent_name`과 `prompt` 파라미터를 받는 `invoke_agent` 도구를 통해 서브에이전트를 디스패치합니다. 동일한 디스패치 작업을 채팅 문법 단축키로도 실행할 수 있습니다. 즉, `@generalist <prompt>`라고 입력하는 것은 `agent_name: "generalist"`로 `invoke_agent`를 호출하는 것과 같습니다. 내장 에이전트 이름으로는 `generalist`, `cli_help`, `codebase_investigator`, 그리고 (브라우저 도구가 활성화된 경우) `browser_agent`가 있습니다.

스킬은 `Subagent (general-purpose):` 형식으로 디스패치하며, 프롬프트 템플릿 파일(예: `superpowers:subagent-driven-development`의 `./implementer-prompt.md`)을 참조하거나 인라인 프롬프트를 제공합니다. Gemini CLI에서의 처리 방식은 다음과 같습니다:

| 스킬 디스패치 형태 | Gemini CLI 대응 방식 |
|---------------------|----------------------|
| `*-prompt.md` 템플릿 참조 (implementer, task-reviewer, code-reviewer 등) | 템플릿의 빈칸을 채운 뒤, `agent_name: "generalist"`와 완성된 프롬프트로 `invoke_agent` 호출 |
| `superpowers:requesting-code-review`의 `./code-reviewer.md` 참조 | `agent_name: "generalist"`와 완성된 리뷰 템플릿으로 `invoke_agent` 호출 |
| 인라인 프롬프트 (참조된 템플릿 없음) | `agent_name: "generalist"`와 인라인 프롬프트로 `invoke_agent` 호출 |

### 프롬프트 채우기 (Prompt filling)

스킬은 `{WHAT_WAS_IMPLEMENTED}` 또는 `[FULL TEXT of task]`와 같은 플레이스홀더가 포함된 프롬프트 템플릿을 제공합니다. 완성된 프롬프트를 `invoke_agent`에 전달하기 전에 모든 플레이스홀더를 실제 내용으로 채우세요. 프롬프트 템플릿 자체에 에이전트의 역할, 리뷰 기준, 기대 출력 형식이 포함되어 있으므로 서브에이전트가 이를 따르게 됩니다.

### 병렬 디스패치 (Parallel dispatch)

Gemini CLI는 병렬 서브에이전트 디스패치를 지원합니다. 독립적인 서브에이전트 작업들을 병렬로 실행하려면 동일한 응답 내에서 여러 개의 `invoke_agent` 호출을 발행하세요(또는 하나의 프롬프트에서 여러 번의 `@generalist` 호출 사용). 의존성이 있는 작업들은 순차적으로 유지하되, 단지 히스토리를 단순하게 유지하겠다는 이유로 독립적인 서브에이전트 작업들을 직렬화(serialize)하지 마세요.

## 추가 Gemini CLI 도구 (Additional Gemini CLI tools)

다음은 Gemini CLI에만 있는 고유 도구들입니다:

| 도구 | 용도 |
|------|---------|
| `save_memory` (레거시) | `experimental.memoryV2 = false`일 때 세션 간에 사실(facts)을 영구 저장 |
| `get_internal_docs` | Gemini CLI에 번들링된 내부 문서 조회 |
| `ask_user` | 사용자에게 구조화된 질문하기 (텍스트 / 단일 선택 / 다중 선택) |
| `enter_plan_mode` / `exit_plan_mode` | 읽기 전용 계획 모드(plan mode)로 진입 및 종료 |
| `update_topic` | 현재 대화의 주제(topic) / 전략적 의도(strategic-intent) 메타데이터 업데이트 |
| `complete_task` | Gemini 서브에이전트가 작업을 완료했음을 알리고 부모 에이전트에게 결과 반환 |
| `tracker_create_task`, `tracker_update_task`, `tracker_get_task`, `tracker_list_tasks`, `tracker_add_dependency`, `tracker_visualize` | 의존성 관리 및 시각화를 지원하는 풍부한 기능의 태스크 트래커 |
| `read_mcp_resource`, `list_mcp_resources` | MCP 리소스 접근 |
