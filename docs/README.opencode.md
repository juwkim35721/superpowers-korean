# OpenCode용 Superpowers

[OpenCode.ai](https://opencode.ai)에서 Superpowers를 사용하기 위한 전체 가이드입니다.

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

### 기존 심볼릭 링크 기반 설치에서 마이그레이션하기 (V1)

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

### 스킬 찾기

OpenCode의 네이티브 `skill` 도구를 사용하여 사용 가능한 모든 스킬의 목록을 확인하세요:

```
use skill tool to list skills
```

### 스킬 로드하기

```
use skill tool to load brainstorming
```

### 개인용 스킬 (Personal Skills)

`~/.config/opencode/skills/`에 나만의 스킬을 생성하세요:

```bash
mkdir -p ~/.config/opencode/skills/my-skill
```

`~/.config/opencode/skills/my-skill/SKILL.md` 파일을 생성하세요:

```markdown
---
name: my-skill
description: Use when [condition] - [what it does]
---

# My Skill

[여기에 스킬 내용을 작성하세요]
```

### 프로젝트 스킬 (Project Skills)

프로젝트 내의 `.opencode/skills/`에 프로젝트 전용 스킬을 생성하세요.

**V2 스킬 우선순위:** 프로젝트 스킬 > 개인용 스킬 > Superpowers 스킬. 테스트된 V1 1.18.31에서는 개인용 또는 프로젝트 스킬이 동일한 이름을 가질 때 번들링된 Superpowers 스킬이 우선 적용되므로, 개인용 및 프로젝트 스킬에는 고유한 이름을 사용하세요. 이 동작은 마이그레이션 이후에도 동일합니다.

## 업데이트 (Updating)

OpenCode는 git 기반 패키지 스펙을 통해 Superpowers를 설치합니다. 일부 OpenCode 및 Bun 버전은 확인된(resolved) git 의존성을 락파일(lockfile)이나 캐시에 고정(pin)하므로, 재시작만으로는 최신 Superpowers 커밋이 반영되지 않을 수 있습니다. 업데이트가 나타나지 않으면 OpenCode의 패키지 캐시를 지우거나 플러그인을 재설치하세요.

특정 버전을 고정하려면 스펙에 태그나 커밋을 추가하세요(V1의 `plugin` 키와 V2의 `plugins` 키 모두 형식은 동일합니다):

```json
{
  "plugin": ["superpowers@git+https://github.com/obra/superpowers.git#v7.0.2"]
}
```

V2에서는 `v6.4.1` 이상을 고정하세요. `v6.3.0` 및 이전 릴리스는 V1에서만 로드됩니다.

## 작동 방식 (How It Works)

플러그인은 호스트 버전별 API를 사용하여 두 가지 작업을 수행합니다:

1. **스킬 디렉토리를 등록하여** 심볼릭 링크나 수동 설정 없이도 OpenCode가 모든 superpowers 스킬을 탐색할 수 있도록 합니다.
    - **V1:** `config` 훅을 통해 `config.skills.paths`에 주입
    - **V2:** `ctx.skill.transform()`을 사용하는 `setup()` 함수를 통해 등록 (V2 네이티브 API, 런타임 작동 확인됨)
2. 버전별 도구 매핑과 함께 **부트스트랩 컨텍스트를 주입합니다**: V1 세션은 아래의 V1 도구 이름을 받고, V2 세션은 V2 이름을 받습니다.
    - **V1:** `experimental.chat.messages.transform` 훅을 통해 주입
    - **V2:** `context`, `compaction`, `generate`에 대한 `ctx.session.hook()`을 통해 주입 — V2의 대응 기능입니다(런타임 작동 확인됨). 세 가지 모두에 등록함으로써 컴팩션(compaction) 요청이 일반 턴과 동일한 캐시된 프롬프트 접두사(prefix)를 유지하도록 합니다.

컨트롤러(controller) 세션은 일시적인(transient) 모델 컨텍스트로 `using-superpowers` 부트스트랩을 받습니다. 위임된 자식(child) 세션은 네이티브 스킬에 대한 접근 권한은 유지하지만 컨트롤러 부트스트랩은 받지 않습니다. 부모 세션이 없는 수동 포크(fork)는 컨트롤러 동작을 유지합니다. V2 네이티브 컴팩션이 이전 사용자 메시지를 유지하는 경우(기본 `compaction.keep.tokens` 예산 범위 내), 컴팩션되지 않은 세션에서와 마찬가지로 체크포인트 앞의 첫 번째 유지된 사용자 메시지에 부트스트랩이 들어갑니다. 컴팩션으로 모든 사용자 메시지가 제거된 경우, 플러그인은 체크포인트 뒤에 일시적인 부트스트랩 메시지를 추가합니다. 어느 쪽이든 저장된 히스토리는 변경되지 않습니다.

세션 조회가 실패하면 플러그인은 해당 요청에 대해 부트스트랩을 유지하고 다음 요청에서 다시 시도합니다. 실패한 조회는 컨트롤러 판정 결과로 캐시되지 않습니다.

### 도구 매핑 (Tool Mapping)

스킬은 특정 런타임의 도구 이름을 직접 지칭하는 대신 수행할 동작(action)으로 서술됩니다. 부트스트랩은 이러한 동작을 사용 중인 OpenCode 버전이 실제로 노출하는 도구에 매핑합니다.

**V1 (`opencode` 1.x):**

- "Create a todo" / "mark complete in todo list" → `todowrite`
- `Subagent (general-purpose):` 템플릿 → `subagent_type: "general"`(또는 코드베이스 탐색의 경우 `"explore"`)을 사용하는 OpenCode의 `task` 도구
- "Invoke a skill" → 순수 스킬 이름을 사용하는 OpenCode의 네이티브 `skill` 도구 (`superpowers:brainstorming` → `name: "brainstorming"`)
- "Read a file" → `read`
- "Create a file" / "edit a file" / "delete a file" → `apply_patch`
- "Run a shell command" → `bash`
- "Search file contents" / "find files by name" → `grep`, `glob`
- "Fetch a URL" → `webfetch`

**V2 (`opencode` 2.0.4 이상; `opencode2`가 별칭으로 제공될 수 있음):**

- "Create a todo" → V2에는 어떠한 종류의 할 일(todo) 도구도 없습니다. 매핑은 모델에게 마크다운 파일(또는 하네스의 계획 기능)에서 계획을 추적하도록 안내합니다
- `Subagent (general-purpose):` 템플릿 → `agent: "general"`(또는 `"explore"`)을 사용하는 OpenCode의 `subagent` 도구; 이전 서브에이전트를 이어서 실행하려면 `sessionID`를 전달하세요
- "Invoke a skill" → 순수 스킬 ID를 사용하는 OpenCode의 네이티브 `skill` 도구 (`superpowers:brainstorming` → `id: "brainstorming"`)
- "Read a file" → `read`
- "Create, edit, or delete files" → 사용 가능한 경우 `patchText`와 함께 `patch`를 사용하세요. 그렇지 않으면 파일 생성 및 덮어쓰기에는 `write`, 특정 부분 수정에는 `edit`, 삭제에는 `shell`을 사용하세요
- "Run a shell command" → `shell` (`command`, `workdir`, `timeout`, `background`)
- "Search file contents" / "find files by name" → `grep`, `glob`
- "Fetch a URL" → `webfetch`
- "Search the web" → `websearch`

요약하면, V2에서는 `task` → `subagent`(에이전트 이름이 `subagent_type`에서 `agent`로 이동했고, 이어서 실행하기 위해 `sessionID`와 함께 재호출함), `apply_patch` → `patch`, `bash` → `shell`로 이름이 변경되었으며, 할 일(todo) 도구는 완전히 제거되었습니다. 사용 가능한 파일 변경(mutation) 도구는 선택된 모델에 따라 다릅니다: 특정 GPT 모델 ID에서는 `patch`를 사용할 수 있으며, 다른 모델들은 `write`와 `edit`를 사용합니다.

(V1 목록은 설치된 OpenCode 1.18.x CLI의 도구 인벤토리를 기준으로 검증되었으며, V2 목록은 OpenCode 2.0.4 및 2.0.7 호스트 규약을 기준으로 검증되었습니다.)

## 문제 해결 (Troubleshooting)

### 플러그인이 로드되지 않는 경우

**V1:** OpenCode 로그를 확인하세요:

```
opencode run --print-logs "hello" 2>&1 | grep -i superpowers
```

**V2:** 플러그인은 백그라운드 서버에서 로드되며, `--print-logs`는 `--standalone` 옵션과 함께 사용할 때만 해당 로그를 표시합니다:

```
opencode run --standalone --print-logs "hello" 2>&1 | grep -i superpowers
```

또는 `~/.local/share/opencode/log/opencode.log`에서 `role=server`로 필터링하여 확인하세요.

또한 `opencode.json`의 플러그인 경로가 올바른지, 최신 버전의 OpenCode를 실행 중인지 확인하세요.

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

1. OpenCode의 `skill` 도구를 사용해 사용 가능한 스킬 목록을 확인하세요.
2. 플러그인이 로드되고 있는지 확인하세요(위 내용 참조).
3. 각 스킬에는 유효한 YAML 프론트매터(frontmatter)가 포함된 `SKILL.md` 파일이 필요합니다.

### 부트스트랩이 나타나지 않는 경우

- **V1:** OpenCode 버전이 `experimental.chat.messages.transform` 훅을 지원하는지 확인하세요. 설정 변경 후에는 OpenCode를 재시작하세요.
- **V2:** 플러그인은 부트스트랩 주입을 위해 `context`, `compaction`, `generate`에 대한 `ctx.session.hook()`을 사용합니다. `opencode api get /api/plugin`을 통해 플러그인이 로드되었는지 확인하세요. 설정 변경 후에는 `opencode service restart`로 재시작하세요. `opencode2` 명령어가 별칭으로 제공될 수도 있습니다.

## 도움 받기 (Getting Help)

- 이슈 제보: https://github.com/obra/superpowers/issues
- 메인 문서: https://github.com/obra/superpowers
- OpenCode V2 문서: https://opencode.ai/v2/docs/
- OpenCode V1 문서: https://opencode.ai/docs/
