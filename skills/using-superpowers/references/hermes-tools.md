# Hermes Agent 도구 매핑 (Tool Mapping)

스킬들은 동작 단위로 표현합니다("서브에이전트 디스패치", "할 일(todo) 생성", "파일 읽기"). Hermes Agent에서 이러한 동작들은 아래 도구들로 매핑됩니다.

## 도구 (Tools)

| 스킬이 요청하는 동작 | Hermes 도구 |
|---|---|
| 파일 읽기 | `read_file` |
| 새 파일 생성 | `write_file` |
| 파일 편집 (특정 부분 패치) | `patch` |
| 셸 명령어 실행 | `terminal` |
| 파일 내용 검색 | `search_files` |
| 이름으로 파일 찾기 | `find`를 사용한 `terminal` |
| URL 가져오기 / 웹페이지 읽기 | `web_extract(urls=[...])` |
| 웹 검색 | `web_search(query=...)` |
| 서브에이전트 디스패치 | `delegate_task(goal=..., context=..., toolsets=[...], role="leaf")` |
| 작업 추적 | `todo` 도구 |
| 스킬 호출 | `skill_view("skill-name")` |

## 지침 파일 (Instructions file)

스킬에서 "지침 파일(your instructions file)"을 언급할 때, Hermes Agent에서 이는 프로젝트 디렉터리의 **`AGENTS.md`**, 또는 전역 경로인 `~/.hermes/SOUL.md`의 **`SOUL.md`**를 가리킵니다.

## 스킬 호출하기 (Invoking a skill)

Hermes Agent에는 `skill_view`와 `skills_list` 도구가 포함된 `skills` 도구 세트(toolset)가 있습니다.
Superpowers 스킬을 호출하려면 다음을 사용하세요:

```
skill_view("brainstorming")
skill_view("test-driven-development")
```

만약 `skill_view`가 Superpowers 스킬을 찾지 못한다면(플러그인이 스킬을 완전히 등록하기 전에는 카탈로그에 나타나지 않을 수 있음), 폴백(fallback)으로 `SKILL.md`를 직접 읽으세요:

```
read_file(path="~/.hermes/plugins/superpowers/skills/<skill-name>/SKILL.md")
```

이 폴백 방식은 네이티브 스킬 로딩 기능이 없는 다른 하네스들에서 사용하는 메커니즘과 동일합니다.

## 서브에이전트 디스패치 (Subagent dispatch)

병렬 또는 순차적 작업 흐름을 위해 격리된 서브에이전트를 생성할 때는 `delegate_task`를 사용하세요:

```
delegate_task(goal="...", context="...", toolsets=[...], role="leaf")
```

`delegate_task`를 사용할 수 없는 경우, 존재하지 않는 도구 호출을 지어내지 말고 현재 세션에서 직접(inline) 작업을 수행하세요.

## 작업 추적 (Task tracking)

세션 내 작업 추적에는 `todo` 도구를 사용하세요. 멀티 에이전트 태스크 보드가 필요한 경우 사용할 수 있다면 `hermes kanban` CLI를 사용하세요. 구버전 문서에 나오는 `TodoWrite` 언급은 위의 작업 추적 동작으로 취급하면 됩니다.
