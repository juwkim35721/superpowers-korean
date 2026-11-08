# Muse 도구 매핑 (Tool Mapping)

스킬들은 동작 단위로 표현합니다("서브에이전트 디스패치", "할 일(todo) 생성", "파일 읽기"). Muse에서 이러한 동작들은 아래 도구들로 매핑됩니다.

| 스킬이 요청하는 동작 | Muse 대응 도구 |
|----------------------|----------------|
| 파일 읽기 | `read_file` |
| 여러 파일 읽기 | `read_file` (여러 번 호출) 또는 `search` |
| 새 파일 생성 | `write_file` |
| 파일 편집 | `edit_file` |
| 셸 명령어 실행 | `bash` |
| 파일 내용 검색 | `search` |
| 이름으로 파일 찾기 | `glob`을 사용한 `search` |
| URL 가져오기 | `web_fetch` |
| 웹 검색 | `web_search` |
| 스킬 호출 | `skills/<name>/SKILL.md`에 대해 `read_file` 실행 또는 네이티브 스킬 도구 사용 |
| 서브에이전트 디스패치 (`Subagent (general-purpose):` 템플릿) | 프롬프트를 채운 후 `subagent_spawn` 호출 |
| 작업 추적 ("할 일(todo) 생성", "완료 표시") | `write_todos` 또는 `bash` 태스크 파일 |
| 사용자에게 질문하기 | `request_user_input` |

## 지침 파일 (Instructions file)

스킬에서 "지침 파일(your instructions file)"을 언급할 때, Muse에서 이는 프로젝트 루트의 **`CLAUDE.md`** 또는 **`AGENTS.md`**를 가리킵니다. Muse는 설정된 경우 이러한 파일들을 계층적으로 로드합니다.

## 스킬 호출 (Skill invocation)

Muse는 `muse skills`를 통해 네이티브 스킬 지원을 제공합니다. Superpowers 스킬을 호출하려면 해당 스킬의 `SKILL.md`를 읽고 지침을 따르세요. 부트스트랩(`using-superpowers`)은 플러그인 훅(hook)을 통해 `SessionStart` 시점에 자동으로 주입됩니다. 이미 적용되어 있으므로 다시 로드하지 마세요.

## 서브에이전트 디스패치 (Subagent dispatch)

격리된 서브에이전트에게 작업을 위임할 때는 `subagent_spawn`을 사용하세요. 디스패치하기 전에 프롬프트 템플릿(예: `implementer-prompt.md`, `task-reviewer-prompt.md`)의 빈칸을 모두 채우세요. 서브에이전트 도구를 사용할 수 없는 경우, 존재하지 않는 도구 호출을 지어내지 말고 현재 세션에서 직접(inline) 작업을 수행하세요.

## 작업 추적 (Task tracking)

체크리스트 추적에는 `write_todos`를 사용하세요. 스킬 체크리스트 항목마다 하나의 할 일(todo)을 생성하고 진행에 따라 `in_progress`/`completed`로 표시하세요. `write_todos`를 사용할 수 없는 경우 `write_file`/`edit_file`을 통해 마크다운 태스크 파일을 유지하세요.
