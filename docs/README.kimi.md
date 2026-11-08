# Kimi Code용 Superpowers

[Kimi Code](https://github.com/MoonshotAI/kimi-code)에서 Superpowers를 사용하기 위한 전체 가이드입니다.

## 설치 (Installation)

Superpowers는 Kimi Code의 플러그인 마켓플레이스에서 사용할 수 있습니다.

플러그인 매니저를 엽니다:

```text
/plugins
```

`Marketplace` > `Superpowers`로 이동하여 설치하세요.

이 저장소에서 직접 설치할 수도 있습니다:

```text
/plugins install https://github.com/obra/superpowers
```

출시 전 `dev` 브랜치를 대상으로 검증하려면 브랜치를 명시적으로 고정하세요:

```text
/plugins install https://github.com/obra/superpowers/tree/dev
```

Kimi Code는 플러그인 변경 사항을 새 세션에 적용합니다. 플러그인을 설치, 업데이트, 활성화, 비활성화 또는 다시 로드한 후에는 `/new`로 새 세션을 시작하세요.

## 작동 방식 (How It Works)

Kimi 플러그인 매니페스트는 `.kimi-plugin/plugin.json`에 위치합니다.

매니페스트는 세 가지 역할을 수행합니다:

1. Kimi Code가 기존 `skills/` 디렉토리를 가리키도록 합니다.
2. `sessionStart.skill`을 통해 세션 시작 시 `using-superpowers`를 로드합니다.
3. `skillInstructions`를 통해 Kimi 전용 도구 매핑을 제공합니다.

Kimi Code는 이 저장소에서 Superpowers 스킬을 직접 읽습니다. 복사된 스킬, 심볼릭 링크, 훅(hooks) 또는 추가 런타임 의존성이 없습니다.

## 도구 매핑 (Tool Mapping)

스킬은 특정 런타임의 도구 이름을 하드코딩하는 대신 수행할 동작(action)을 설명합니다. Kimi Code에서 이러한 동작은 다음과 같이 매핑됩니다:

- "Ask the user" / "ask clarifying questions" -> `AskUserQuestion`
- "Create a todo" / "mark complete in todo list" -> `TodoList`
- "Dispatch a subagent" -> `Agent`
- "Invoke a skill" -> Kimi Code의 네이티브 `Skill` 도구
- "Read a file" / "write a file" / "edit a file" -> `Read`, `Write`, `Edit`
- "Run a shell command" -> `Bash`
- "Search file contents" -> `Grep`
- "Find files by path or pattern" -> `Glob`
- "Fetch a URL" -> `FetchURL`
- "Search the web" -> `WebSearch`

## 업데이트 (Updating)

Kimi Code의 플러그인 매니저를 사용하세요:

```text
/plugins
```

Superpowers를 선택하고 그곳에서 업데이트하세요. 업데이트 후에는 `/new`로 새 세션을 시작하세요.

## 문제 해결 (Troubleshooting)

### 플러그인이 로드되지 않는 경우

1. `/plugins info superpowers`를 실행하고 진단 정보를 확인하세요.
2. 플러그인이 활성화(enabled)되어 있는지 확인하세요.
3. 설치 또는 업데이트 후 `/new`로 새 세션을 시작하세요.

### GitHub 직접 설치 시 이전 릴리스가 설치된 경우

Kimi Code는 기본 저장소 URL이 주어지면 최신 GitHub 릴리스(존재하는 경우)를 설치합니다. 다음 Superpowers 릴리스 전에 미출시 변경 사항을 테스트하려면 브랜치를 명시적으로 지정하여 설치하세요:

```text
/plugins install https://github.com/obra/superpowers/tree/dev
```

### 스킬이 트리거되지 않는 경우

1. `/plugins info superpowers`에서 플러그인이 활성화되어 있는지 확인하세요.
2. `/new`로 새 세션을 시작하세요.
3. 인수 테스트 프롬프트를 시도해 보세요: `Let's make a react todo list`. 정상적으로 설치되었다면 코드를 작성하기 전에 `brainstorming`을 로드해야 합니다.
