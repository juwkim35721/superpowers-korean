# OpenCode용 Superpowers 설치하기

## 사전 요구사항 (Prerequisites)

- [OpenCode.ai](https://opencode.ai) 설치됨

## 설치 (Installation)

OpenCode V2는 버전 2.0.4 이상이 필요합니다.

### OpenCode V1

기존 V1 플러그인 설정을 사용하세요:

```json
{
  "plugin": ["superpowers@git+https://github.com/obra/superpowers.git"]
}
```

### OpenCode V2 (2.0.4 이상)

V2 플러그인 설정을 사용하세요:

```json
{
  "plugins": ["superpowers@git+https://github.com/obra/superpowers.git"]
}
```

로컬 V2 설치의 경우, `index.js`가 포함된 저장소 디렉토리를 설정하세요. OpenCode 2.0.4 및 2.0.7은 직접 지정된 JavaScript 파일 경로 설정을 거부합니다. 탐색된 플러그인 심볼릭 링크(symlink)는 계속 지원됩니다.

OpenCode를 재시작하세요. V2는 `opencode` 명령어를 사용하며, `opencode2`가 별칭(alias)으로 제공될 수도 있습니다. 플러그인은 OpenCode의 플러그인 매니저를 통해 설치되고 모든 스킬을 등록합니다.

다음과 같이 질문하여 설치를 확인하세요: "Tell me about your superpowers"

OpenCode는 자체적인 플러그인 설치 방식을 사용합니다. 만약 Claude Code, Codex 또는 다른 하네스(harness)도 함께 사용한다면 각각에 대해 Superpowers를 별도로 설치하세요.

## 기존 심볼릭 링크 기반 설치에서 마이그레이션하기

이전에 `git clone`과 심볼릭 링크를 사용해 superpowers를 설치했다면 기존 설정을 제거하세요:

```bash
# 기존 심볼릭 링크 제거
rm -f ~/.config/opencode/plugins/superpowers.js
rm -rf ~/.config/opencode/skills/superpowers

# 선택 사항: 클론된 저장소 제거
rm -rf ~/.config/opencode/superpowers

# superpowers를 위해 opencode.json에 추가한 skills.paths가 있다면 제거
```

그런 다음 위의 설치 단계를 따르세요.

## 사용법 (Usage)

OpenCode의 네이티브 `skill` 도구를 사용하세요:

```
use skill tool to list skills
use skill tool to load brainstorming
```

## 업데이트 (Updating)

OpenCode는 git 기반 패키지 스펙을 통해 Superpowers를 설치합니다. 일부 OpenCode 및 Bun 버전은 확인된(resolved) git 의존성을 락파일(lockfile)이나 캐시에 고정(pin)하므로, 재시작만으로는 최신 Superpowers 커밋이 반영되지 않을 수 있습니다. 업데이트가 나타나지 않으면 OpenCode의 패키지 캐시를 지우거나 플러그인을 재설치하세요.

특정 버전을 고정하려면 스펙에 태그나 커밋을 추가하세요(V1의 `plugin` 키와 V2의 `plugins` 키 모두 형식은 동일합니다):

```json
{
  "plugin": ["superpowers@git+https://github.com/obra/superpowers.git#v7.0.2"]
}
```

V2에서는 `v6.4.1` 이상을 고정하세요. `v6.3.0` 및 이전 릴리스는 V1에서만 로드됩니다.

## 문제 해결 (Troubleshooting)

### 플러그인이 로드되지 않는 경우

1. 로그를 확인하세요. V1: `opencode run --print-logs "hello" 2>&1 | grep -i superpowers`.
   V2는 백그라운드 서버에서 플러그인을 로드하므로 `--standalone`을 추가하세요:
   `opencode run --standalone --print-logs "hello" 2>&1 | grep -i superpowers`,
   또는 `~/.local/share/opencode/log/opencode.log`에서 `role=server`로 필터링하여 확인하세요.
2. `opencode.json`의 플러그인 항목을 확인하세요.
3. 최신 버전의 OpenCode를 실행 중인지 확인하세요.

### Windows 설치 문제

일부 Windows용 OpenCode 빌드에는 `git+https` URL의 캐시 경로 문제나 일반 터미널에서는 정상 작동하는 `git.exe`를 Bun이 찾지 못하는 문제 등 git 기반 플러그인 스펙과 관련된 업스트림 설치 관리자 이슈가 있습니다. OpenCode가 플러그인을 설치하지 못한다면 시스템 npm으로 설치한 뒤 OpenCode가 로컬 패키지를 가리키도록 설정해 보세요:

```powershell
npm install superpowers@git+https://github.com/obra/superpowers.git --prefix "$HOME\.config\opencode"
```

그런 다음 사용 중인 OpenCode 버전에 맞게 `opencode.json`에 설치된 패키지의 절대 경로를 지정하세요. OpenCode는 `~`를 확장하지 않으므로 `~/...` 항목은 로컬 디렉토리가 아닌 패키지 이름으로 처리됩니다.

**V1:**

```json
{
  "plugin": ["C:\\Users\\<you>\\.config\\opencode\\node_modules\\superpowers"]
}
```

**V2 (2.0.4 이상):**

```json
{
  "plugins": ["C:\\Users\\<you>\\.config\\opencode\\node_modules\\superpowers"]
}
```

### 스킬을 찾을 수 없는 경우

1. `skill` 도구를 사용해 탐색된 스킬 목록을 확인하세요.
2. 플러그인이 로드되고 있는지 확인하세요(위 내용 참조).

### 도구 매핑 (Tool mapping)

스킬은 동작("할 일 생성(create a todo)", "서브에이전트 디스패치(dispatch a subagent)", "파일 읽기(read a file)") 단위로 서술됩니다. 플러그인은 버전에 맞는 매핑을 주입합니다 — 사용 중인 OpenCode 버전을 확인하세요:

**V1 (`opencode` 1.x):**

- "Create a todo" / "mark complete in todo list" → `todowrite`
- `Subagent (general-purpose):` 템플릿 → `subagent_type: "general"`(또는 코드베이스 탐색의 경우 `"explore"`)을 사용하는 `task` 도구
- "Invoke a skill" → OpenCode의 네이티브 `skill` 도구
- "Read a file" → `read`
- "Create a file" / "edit a file" / "delete a file" → `apply_patch`
- "Run a shell command" → `bash`
- "Search file contents" / "find files by name" → `grep`, `glob`
- "Fetch a URL" → `webfetch`

**V2 (`opencode` 2.0.4 이상; `opencode2`가 별칭으로 제공될 수 있음):**

- "Create a todo" → V2에는 할 일(todo) 도구가 없습니다. 대신 마크다운 파일에서 계획을 추적하세요
- `Subagent (general-purpose):` 템플릿 → `agent: "general"`(또는 `"explore"`)을 사용하는 `subagent` 도구; 이전 서브에이전트를 이어서 실행하려면 `sessionID`를 전달하세요
- "Invoke a skill" → OpenCode의 네이티브 `skill` 도구
- "Read a file" → `read`
- "Create, edit, or delete files" → 사용 가능한 경우 `patchText`와 함께 `patch`를 사용하세요. 그렇지 않으면 파일 생성 및 덮어쓰기에는 `write`, 특정 부분 수정에는 `edit`, 삭제에는 `shell`을 사용하세요
- "Run a shell command" → `shell` (`command`, `workdir`, `timeout`, `background`)
- "Search file contents" / "find files by name" → `grep`, `glob`
- "Fetch a URL" → `webfetch`
- "Search the web" → `websearch`

## 도움 받기 (Getting Help)

- 이슈 제보: https://github.com/obra/superpowers/issues
- 전체 문서: https://github.com/obra/superpowers/blob/main/docs/README.opencode.md
