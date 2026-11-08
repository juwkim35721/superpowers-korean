# Antigravity 마켓플레이스 플러그인

## 목표

Google과의 파트너십의 일환으로 Google Antigravity 플러그인 마켓플레이스에 Superpowers를 등재합니다. 이를 위해 마켓플레이스 표시 필드가 포함된 네이티브 Antigravity 플러그인 매니페스트, 로고, 그리고 네이티브 플러그인 경로에서 작동하는 세션 부트스트랩이 필요합니다.

참고 자료: "Antigravity Plugin Marketplace Publishing Guide" (Google 제공 PDF), 및 아래에 기록된 `agy` 1.3.1 대상 프로브 결과.

## 현재 상태 (`agy` 1.3.1)

- `agy plugin install https://github.com/obra/superpowers`는 성공하지만, 오직 `agy`의 Gemini 익스텐션 폴백(fallback)을 통해서만 작동합니다: `agy plugin list`가 `"source": "gemini-cli"`를 보고합니다. 부트스트랩은 `GEMINI.md`의 `@` 라인에 편승하며, 모델은 이를 힌트로 취급하여 `view_file`로 따라가 읽습니다. 이 경로에서 인수 테스트(acceptance test)는 통과합니다.
- `agy`는 Claude 포맷인 `hooks/hooks.json`을 설치 루트에 `hooks.json`으로 복사하고 매 세션마다 파싱에 실패합니다:
  `invalid hook "hooks": command hook must specify 'command'`.
- 저장소에서 `agy plugin validate .`를 실행하면 실패합니다: `missing plugin.json`.

## 프로브 발견 사항 (`agy` 1.3.1)

각 프로브는 일회용 플러그인을 설치하고 `agy -p`를 실행한 뒤 삭제하는 방식으로 진행했습니다.

1. `agy`의 훅 이벤트는 `PreToolUse`, `PostToolUse`, `PreInvocation`, `PostInvocation`, `Stop`입니다. `SessionStart`는 없습니다. `agy`의 내장 문서는 이름이 지정된 최상위 훅(`{"<name>": {"PreToolUse": [...]}}`)을 사용하며, 이는 PDF 가이드의 `{"hooks": {...}}` 예시와 다릅니다.
2. 플러그인 내의 `rules/AGENTS.md`는 매 세션마다 상시 활성(always-on) 규칙으로 로드됩니다.
3. 규칙 내에서 규칙 파일 기준의 상대 `@` 경로는 절대 설치 경로로 재작성됩니다(`@../skills/x/SKILL.md`가 `@/Users/<u>/.gemini/config/plugins/<plugin>/skills/x/SKILL.md`가 됨). 파일 내용은 인라인으로 삽입되지 않습니다. 규칙 내에서는 `${PLUGIN_ROOT}`가 치환되지 않습니다. 규칙 파일 기준으로 확인되지 않는 경로는 작성된 그대로 남습니다.
4. `.antigravity-plugin/plugin.json`이 존재하면 `agy`는 플러그인을 `"source": "antigravity"`로 설치하고 15개 스킬을 모두 찾으며, 루트 `hooks.json`이 없으므로 훅을 건너뛰어 파싱 경고가 사라집니다.
5. 네이티브 매니페스트와 한 줄짜리 `rules/AGENTS.md`(`@../skills/using-superpowers/SKILL.md`)를 갖춘 저장소 스테이징 복사본이 인수 테스트를 통과했습니다: 모델은 `using-superpowers`를 읽은 뒤 `antigravity-tools.md`를 읽고, 이어서 `brainstorming`을 읽은 다음 코드 작성 없이 명확화 질문과 함께 "Using superpowers:brainstorming…"이라고 응답했습니다.
6. 저장소 루트의 `AGENTS.md`(기여자 가이드라인)는 설치 경로로 복사되지만 규칙으로 로드되지는 않습니다. 오직 `rules/AGENTS.md`만 규칙으로 로드됩니다.

## 설계

### 1. 네이티브 매니페스트: `.antigravity-plugin/plugin.json`

가이드는 `.antigravity-plugin/plugin.json` 위치의 매니페스트를 허용하며, 이는 우리의 다른 하네스 숨김 디렉터리(dotdirs) 구조와도 일치합니다. 필드 구성:

| 필드 | 값 |
|---|---|
| `name` | `superpowers` |
| `displayName` | `Superpowers` |
| `version` | 현재 릴리스 버전, `.version-bump.json`으로 관리됨 |
| `description` | `Make your coding agent a better engineer` (Jesse가 작성한 문구; 가이드 권장 길이인 120–160자보다 짧으며 최대 160자 제한 충족) |
| `logo` | `assets/antigravity-logo.png` |
| `suggestedPrompts` | `I've got an idea for something I'd like to build`, `Let's figure out why this broke` (Jesse가 작성한 문구) |
| `category` | `Developer Tools` |
| `keywords` | 소문자, `.claude-plugin/plugin.json`의 키워드와 일치 |
| `author` | `{ "name": "Jesse Vincent", "email": "jesse@fsck.com" }`, `.claude-plugin/plugin.json`과 일치 |
| `homepage`, `repository` | `https://github.com/obra/superpowers` |
| `license` | `MIT` |

`skills`나 `mcpServers` 필드는 생략합니다: 기본값(`skills/`, MCP 설정 없음)이 우리가 원하는 바이며, 프로브 4에서 기본 탐색 기능이 모든 스킬을 찾아냄을 확인했습니다.

### 2. 부트스트랩: 추가 파일 없이 스킬 탐색 활용

`agy`는 설치된 각 스킬의 설명을 나열하며, `using-superpowers`의 "Use when starting any conversation" 설명이 모델로 하여금 해당 스킬을 로드하게 만듭니다. 플러그인은 `rules/`도 `hooks.json`도 배포하지 않습니다.

이는 최종 리뷰에서 동일한 저장소 루트에서 플러그인이 배포하는 모든 `rules/*.md`를 Cursor 또한 로드한다는 사실을 발견한 후, 한 줄짜리 `rules/AGENTS.md` 포인터를 사용하려던 첫 번째 설계를 대체한 것입니다. `agy` 1.3.1에서 프로브해 보았으나 기각된 대안들:

- `PreInvocation` 훅. 모든 모델 호출 전에 실행되며 `hooks.json`이 반드시 플러그인 루트에 위치해야 합니다. `ephemeralMessage`는 다음 호출에서 버려지고, `userMessage`는 유지되므로 `invocationNum == 0` 및 `initialNumSteps == 1` 조건으로 게이트를 걸면 대화당 정확히 한 번만 주입되지만, 여전히 `agy`가 매 호출마다 훅 프로세스를 시작하는 오버헤드가 발생합니다.
- `hooks.json`이나 `rules/`의 위치를 변경해 주는 매니페스트 필드는 존재하지 않습니다.

부트스트랩 파일 없이 12/12회 실행 검증 완료: "Let's make a react todo list"(3회)와 "add a settings page"(3회)는 코드를 작성하기 전에 `brainstorming`을 로드했고, "this test is failing, can you fix it?"(3회)은 편집 전에 `systematic-debugging`을 로드했으며, 관련 없는 첫 번째 턴 이후 두 번째 턴에서 react-todo를 요청한 경우(3회)에도 `brainstorming`을 로드했습니다. 모델: Gemini 3.8 Flash. 위험 요소는 트리거링이 스킬 설명을 준수하는 모델의 동작에 의존한다는 점이며, 만약 이것이 흔들릴 경우 대화당 1회 주입하는 훅 방식이 폴백 대안입니다.

### 3. 로고: `assets/antigravity-logo.png`

투명 배경의 512×512 PNG입니다. 흰색 둥근 정사각형(안쪽 여백 31px, 모서리 반경 100) 안에 `#1a1a1a` 색상의 `assets/superpowers-small.svg` 마크가 정사각형 너비의 약 70% 크기로 중앙에 배치됩니다. 테두리는 없으며, 라이트 테마에서는 흰색 바탕 위의 마크만으로 보이는데 이는 허용 가능한 수준입니다.

`scripts/render-antigravity-logo.sh`가 `superpowers-small.svg`로부터 SVG를 합성하고 `rsvg-convert`로 래스터화합니다. 이 스크립트가 PNG를 재생성하는 유일한 경로이며, 도움말 텍스트를 갖추고 있고 `rsvg-convert`가 없을 때 명확한 에러와 함께 실패합니다.

### 4. 버전 업데이트 관리

`.version-bump.json`에 `{ "path": ".antigravity-plugin/plugin.json", "field": "version" }`을 추가합니다.

### 5. Codex 동기화

`scripts/sync-to-codex-plugin.sh`의 `EXCLUDES`에 `/.antigravity-plugin/`과 `/assets/antigravity-logo.png`를 추가하고, `tests/codex-plugin-sync/test-sync-to-codex-plugin.sh`에서 이를 검증합니다.

### 6. README

Antigravity 설치 명령어는 그대로 `agy plugin install https://github.com/obra/superpowers`로 유지합니다. 설명 문구는 "플러그인의 세션 시작 훅을 실행한다"에서 스킬 탐색을 설명하는 내용으로 변경합니다. 마켓플레이스 경로는 실제로 개설된 후에 추가하며, 그전까지는 관련 주장을 적지 않습니다.

### 7. 테스트: `tests/antigravity/`

- `test-plugin-manifest.sh` (CI 안전, `tests/kimi/test-plugin-manifest.sh` 구조를 미러링함):
  - 매니페스트가 정상 파싱됨; `name`이 케밥 케이스 패턴과 일치하며 `superpowers`와 같음;
  - 마켓플레이스 필수 필드가 모두 존재하고 비어 있지 않음: `name`, `description`, `displayName`, `logo`, `suggestedPrompts`(1–3개의 비어 있지 않은 문자열), `version`, `author`;
  - `description`이 최대 160자 이하임;
  - `version`이 `.claude-plugin/plugin.json`의 버전과 일치함;
  - `logo` 파일이 존재하고, PNG 포맷이며, 정사각형이고 최소 128×128 이상임;
  - `.version-bump.json`에 매니페스트가 등재되어 있음;
  - 부트스트랩이 의존하는 `using-superpowers`의 설명에 여전히 "Use when starting any conversation"이 포함되어 있음;
  - 루트에 `plugin.json`, `hooks.json`, 또는 `rules/`가 존재하지 않음.
- `PATH`에 `agy`가 있는 경우 저장소에 대해 `agy plugin validate`를 추가로 실행하고 스킬 처리 완료와 함께 `[ok]`가 출력되는지 확인합니다.
- 여전히 `agy`가 SessionStart 훅을 실행한다고 적혀 있는 `test-antigravity-tools.sh`의 헤더 주석을 업데이트합니다.

실제 환경 인수 테스트 (CI에는 미포함): 브랜치의 트리를 `agy`에 설치하고, 깨끗한 워크스페이스에서 "Let's make a react todo list"를 최소 세 번 이상 실행하여 매번 코드가 작성되기 전에 `brainstorming`이 트리거되는지 확인합니다. 테스트 후에는 플러그인을 삭제합니다.

### 8. 포팅 가이드: 별도 커밋으로 진행

`docs/porting-to-a-new-harness.md`는 실제로 존재하지 않는 `.antigravity-plugin/install.sh`와 생성된 `ANTIGRAVITY.md`를 설명하고 있습니다. 이를 별도의 커밋에서 수정합니다:

- 라우팅 표 행: Antigravity는 노출된 스킬 인덱스를 사용함.
- "no `Skill` tool" 섹션: Antigravity에 대한 `contextFileName` 조언을 스킬 인덱스 부트스트랩으로 교체하고, 훅 및 규칙 관련 발견 사항(세션 시작 이벤트 없음, 호출마다 실행되는 `PreInvocation`, 훅 내 `${PLUGIN_ROOT}` 미지원, 공유 플러그인 루트가 Cursor로 `rules/`를 유출함)을 기록함.
- 배포 표: Antigravity는 git URL(`agy plugin install <url>`)로부터 설치하며 자체 마켓플레이스에 등재됨.
- 설치 스크립트 제거(installer-stripping) 섹션: `install.sh` / `ANTIGRAVITY.md` 예시 제거.
- 하네스 인덱스 표에 Antigravity 행이 없으므로 하나 추가함(매니페스트, 부트스트랩, 도구 매핑, 테스트, 설치).

## 범위 외 (Out of scope) 항목

- 마켓플레이스 제출 절차. 가이드에는 제출 방법이 나와 있지 않으며, 이는 Jesse의 Google 담당자를 통해 진행됩니다.
- Antigravity 2.0 및 IDE. 현재 이곳에 설치되어 있지 않아 검증되지 않았으므로 README에서는 이에 대해 어떠한 주장도 하지 않습니다.
- `using-superpowers`를 생성된 규칙으로 복사하는 열려 있는 PR #2350. 이 작업이 반영된 후 어떻게 응답할지는 Jesse가 결정하며, 답변 초안은 제가 작성합니다.
- Codex 동기화 제외 목록에서 `.hermes-plugin/`과 `.muse-plugin/`이 누락된 문제. 이번 작업과 무관하므로 별도의 수정을 위해 기록만 해둡니다.
