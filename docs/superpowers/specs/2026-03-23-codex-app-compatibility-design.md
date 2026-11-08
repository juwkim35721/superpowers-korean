# Codex App 호환성: 워크트리 및 마무리 스킬 적응 (Codex App Compatibility: Worktree and Finishing Skill Adaptation)

기존 Claude Code나 Codex CLI의 동작을 깨뜨리지 않으면서 Codex App의 샌드박스 워크트리 환경에서 superpowers 스킬이 작동하도록 만듭니다.

**티켓:** PRI-823

## 동기 (Motivation)

Codex App은 자체적으로 관리하는 git 워크트리 내부에서 에이전트를 실행합니다 — `$CODEX_HOME/worktrees/` 아래에 위치하며 분리된 HEAD(detached HEAD) 상태이고, `git checkout -b`, `git push` 및 네트워크 접근을 차단하는 Seatbelt 샌드박스가 적용되어 있습니다. 반면 세 가지 superpowers 스킬은 제한 없는 git 접근 권한을 가정합니다: `using-git-worktrees`는 명명된 브랜치로 수동 워크트리를 생성하고, `finishing-a-development-branch`는 브랜치 이름으로 병합/푸시/PR을 수행하며, `subagent-driven-development`는 이 두 가지를 모두 필요로 합니다.

오픈소스 터미널 도구인 Codex CLI에는 이러한 충돌이 없습니다 — 내장된 워크트리 관리 기능이 없기 때문입니다. 그곳에서는 우리의 수동 워크트리 접근 방식이 격리 공백을 메워줍니다. 이 문제는 오직 Codex App에서만 발생합니다.

## 실증적 조사 결과 (Empirical Findings)

2026-03-23 Codex App에서 테스트함:

| 작업 | workspace-write 샌드박스 | Full access 샌드박스 |
|---|---|---|
| `git add` | 작동함 | 작동함 |
| `git commit` | 작동함 | 작동함 |
| `git checkout -b` | **차단됨** (`.git/refs/heads/`에 쓸 수 없음) | 작동함 |
| `git push` | **차단됨** (네트워크 + `.git/refs/remotes/`) | 작동함 |
| `gh pr create` | **차단됨** (네트워크) | 작동함 |
| `git status/diff/log` | 작동함 | 작동함 |

추가 발견 사항:
- `spawn_agent` 서브에이전트는 부모 스레드의 파일시스템을 **공유**합니다 (마커 파일 테스트를 통해 확인됨)
- 워크트리가 어떤 브랜치에서 시작되었는지와 상관없이 App 헤더에 "Create branch" 버튼이 나타납니다
- App의 네이티브 마무리 흐름: Create branch → Commit 모달 → Commit and push / Commit and create PR
- macOS에서 `network_access = true` 설정이 아무런 경고 없이 작동하지 않는 버그가 있습니다 (이슈 #10390)

## 설계: 읽기 전용 환경 감지 (Read-Only Environment Detection)

세 가지 읽기 전용 git 명령어로 부작용 없이 환경을 감지합니다:

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

여기서 두 가지 신호가 도출됩니다:

- **IN_LINKED_WORKTREE:** `GIT_DIR != GIT_COMMON` — 에이전트가 외부(Codex App, Claude Code Agent 도구, 이전 스킬 실행 또는 사용자)에서 생성한 워크트리 내부에 있음
- **ON_DETACHED_HEAD:** `BRANCH`가 비어 있음 — 명명된 브랜치가 존재하지 않음

`show-toplevel`을 확인하는 대신 `git-dir != git-common-dir`를 사용하는 이유:
- 일반 저장소에서는 두 명령어 모두 동일한 `.git` 디렉터리로 해석됩니다
- 연결된(linked) 워크트리에서는 `git-dir`가 `.git/worktrees/<name>`인 반면 `git-common-dir`는 `.git`입니다
- 서브모듈에서는 두 값이 동일하므로, `show-toplevel`이 유발할 수 있는 오탐(false positive)을 방지합니다
- `cd && pwd -P`를 통해 경로를 해석하면 상대 경로 문제(`git-common-dir`는 일반 저장소에서는 상대 경로인 `.git`을 반환하지만 워크트리에서는 절대 경로를 반환함)와 심볼릭 링크 문제(macOS의 `/tmp` → `/private/tmp`)를 모두 해결할 수 있습니다

### 의사결정 매트릭스 (Decision Matrix)

| 연결된 워크트리인가? | Detached HEAD인가? | 환경 | 동작 |
|---|---|---|---|
| 아니요 | 아니요 | Claude Code / Codex CLI / 일반 git | 전체 스킬 동작 (변경 없음) |
| 예 | 예 | Codex App 워크트리 (workspace-write) | 워크트리 생성 건너뛰기; 마무리 시 핸드오프 페이로드 출력 |
| 예 | 아니요 | Codex App (Full access) 또는 수동 워크트리 | 워크트리 생성 건너뛰기; 전체 마무리 흐름 진행 |
| 아니요 | 예 | 비정상적 상황 (수동 detached HEAD) | 정상적으로 워크트리 생성; 마무리 시 경고 |

## 변경 사항

### 1. `using-git-worktrees/SKILL.md` — Step 0 추가 (~12줄)

"Overview"와 "Directory Selection Process" 사이에 새 섹션을 추가합니다:

**Step 0: 이미 격리된 작업 공간에 있는지 확인 (Check if Already in an Isolated Workspace)**

감지 명령어를 실행합니다. `GIT_DIR != GIT_COMMON`이면 워크트리 생성을 완전히 건너뜁니다. 대신 다음을 수행합니다:
1. Creation Steps 아래의 "Run Project Setup" 하위 섹션으로 건너뜁니다 — `npm install` 등은 멱등성(idempotent)이 있으므로 안전을 위해 실행할 가치가 있습니다
2. 그런 다음 "Verify Clean Baseline"을 수행합니다 — 테스트를 실행합니다
3. 브랜치 상태와 함께 보고합니다:
   - 브랜치에 있는 경우: "Already in an isolated workspace at `<path>` on branch `<name>`. Tests passing. Ready to implement."
   - Detached HEAD인 경우: "Already in an isolated workspace at `<path>` (detached HEAD, externally managed). Tests passing. Note: branch creation needed at finish time. Ready to implement."

`GIT_DIR == GIT_COMMON`이면 전체 워크트리 생성 흐름을 진행합니다 (변경 없음).

Step 0가 발동되면 안전성 검증(.gitignore 확인)은 건너뜁니다 — 외부에서 생성된 워크트리에는 무관하기 때문입니다.

Integration 섹션의 "Called by" 항목들을 업데이트합니다. 각 항목의 설명을 컨텍스트 의존적인 텍스트에서 "Ensures isolated workspace (creates one or verifies existing)"으로 변경합니다. 예를 들어, `subagent-driven-development` 항목은 "REQUIRED: Set up isolated workspace before starting"에서 "REQUIRED: Ensures isolated workspace (creates one or verifies existing)"으로 변경됩니다.

**샌드박스 폴백 (Sandbox fallback):** `GIT_DIR == GIT_COMMON`이어서 스킬이 Creation Steps로 진행했지만 `git worktree add -b`가 권한 에러(예: Seatbelt 샌드박스 거부)로 실패하는 경우, 이를 뒤늦게 감지된 제한된 환경으로 처리합니다. Step 0의 "이미 작업 공간에 있음" 동작으로 폴백합니다 — 생성을 건너뛰고 현재 디렉터리에서 설정 및 베이스라인 테스트를 실행한 뒤 그에 맞게 보고합니다.

Step 0에서 보고한 후에는 멈춥니다(STOP). Directory Selection이나 Creation Steps로 계속 진행하지 마세요.

**그 외 모든 것은 변경 없음:** Directory Selection, Safety Verification, Creation Steps, Project Setup, Baseline Tests, Quick Reference, Common Mistakes, Red Flags.

### 2. `finishing-a-development-branch/SKILL.md` — Step 1.5 및 정리 가드 추가 (~20줄)

**Step 1.5: 환경 감지 (Detect Environment)** (Step 1 "Verify Tests" 이후, Step 2 "Determine Base Branch" 이전)

감지 명령어를 실행합니다. 세 가지 경로가 있습니다:

- **경로 A**는 Step 2와 Step 3를 완전히 건너뜁니다 (베이스 브랜치나 옵션이 필요 없음).
- **경로 B와 C**는 평소처럼 Step 2(Determine Base Branch)와 Step 3(Present Options)를 진행합니다.

**경로 A — 외부 관리 워크트리 + detached HEAD** (`GIT_DIR != GIT_COMMON` 그리고 `BRANCH`가 비어 있음):

먼저 모든 작업이 스테이징되고 커밋되었는지 확인합니다 (`git add` + `git commit`). Codex App의 마무리 컨트롤은 커밋된 작업을 대상으로 작동합니다.

그런 다음 사용자에게 다음 내용을 제시합니다 (4가지 옵션 메뉴는 제시하지 **마세요**):

```
Implementation complete. All tests passing.
Current HEAD: <full-commit-sha>

This workspace is externally managed (detached HEAD).
I cannot create branches, push, or open PRs from here.

⚠ These commits are on a detached HEAD. If you do not create a branch,
they may be lost when this workspace is cleaned up.

If your host application provides these controls:
- "Create branch" — to name a branch, then commit/push/PR
- "Hand off to local" — to move changes to your local checkout

Suggested branch name: <ticket-id/short-description>
Suggested commit message: <summary-of-work>
```

브랜치 이름 도출: 사용 가능한 경우 티켓 ID를 사용하고(예: `pri-823/codex-compat`), 그렇지 않으면 계획 제목의 처음 5개 단어를 슬러그화(slugify)하며, 그마저도 없으면 제안을 생략합니다. 브랜치 이름에 민감한 내용(취약점 설명, 고객 이름 등)을 포함하지 마세요.

Step 5로 건너뜁니다 (외부에서 관리되는 워크트리의 경우 정리 작업은 아무 일도 하지 않는 no-op입니다).

**경로 B — 외부 관리 워크트리 + 명명된 브랜치** (`GIT_DIR != GIT_COMMON` 그리고 `BRANCH`가 존재함):

평소처럼 4가지 옵션 메뉴를 제시합니다. (Step 5 정리 가드가 외부 관리 상태를 독립적으로 다시 감지할 것입니다.)

**경로 C — 일반 환경** (`GIT_DIR == GIT_COMMON`):

현재와 같이 4가지 옵션 메뉴를 제시합니다 (변경 없음).

**Step 5 정리 가드 (cleanup guard):**

정리 시점에 `GIT_DIR` 대 `GIT_COMMON` 감지를 다시 실행합니다 (이전 스킬 출력에 의존하지 마세요 — 마무리 스킬은 다른 세션에서 실행될 수 있습니다). `GIT_DIR != GIT_COMMON`이면 `git worktree remove`를 건너뜁니다 — 호스트 환경이 이 작업 공간을 소유하고 있기 때문입니다.

그렇지 않으면 현재와 같이 확인 후 제거합니다. 참고: 기존 Step 5 텍스트에는 "For Options 1, 2, 4"라고 되어 있지만 Quick Reference 표와 Common Mistakes 섹션에는 "Options 1 & 4 only"라고 되어 있습니다. 새로운 가드는 이 기존 로직 이전에 추가되며 어떤 옵션이 정리를 트리거하는지는 변경하지 않습니다.

**그 외 모든 것은 변경 없음:** Options 1-4 로직, Quick Reference, Common Mistakes, Red Flags.

### 3. `subagent-driven-development/SKILL.md` 및 `executing-plans/SKILL.md` — 각각 1줄 수정

두 스킬 모두 동일한 Integration 섹션 줄을 가지고 있습니다. 다음 내용을:
```
- superpowers:using-git-worktrees - REQUIRED: Set up isolated workspace before starting
```
다음으로 변경합니다:
```
- superpowers:using-git-worktrees - REQUIRED: Ensures isolated workspace (creates one or verifies existing)
```

**그 외 모든 것은 변경 없음:** 디스패치/리뷰 루프, 프롬프트 템플릿, 모델 선택, 상태 처리, Red Flags.

### 4. `codex-tools.md` — 환경 감지 문서 추가 (~15줄)

파일 끝에 두 개의 새 섹션을 추가합니다:

**환경 감지 (Environment Detection):**

```markdown
## Environment Detection

Skills that create worktrees or finish branches should detect their
environment with read-only git commands before proceeding:

\```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
\```

- `GIT_DIR != GIT_COMMON` → already in a linked worktree (skip creation)
- `BRANCH` empty → detached HEAD (cannot branch/push/PR from sandbox)

See `using-git-worktrees` Step 0 and `finishing-a-development-branch`
Step 1.5 for how each skill uses these signals.
```

**Codex App 마무리 (Codex App Finishing):**

```markdown
## Codex App Finishing

When the sandbox blocks branch/push operations (detached HEAD in an
externally managed worktree), the agent commits all work and informs
the user to use the App's native controls:

- **"Create branch"** — names the branch, then commit/push/PR via App UI
- **"Hand off to local"** — transfers work to the user's local checkout

The agent can still run tests, stage files, and output suggested branch
names, commit messages, and PR descriptions for the user to copy.
```

## 변경되지 않는 사항

- `implementer-prompt.md`, `spec-reviewer-prompt.md`, `code-quality-reviewer-prompt.md` — 서브에이전트 프롬프트는 그대로 유지됨
- `executing-plans/SKILL.md` — Integration 설명 1줄만 변경되며(`subagent-driven-development`와 동일), 모든 런타임 동작은 변경되지 않음
- `dispatching-parallel-agents/SKILL.md` — 워크트리나 마무리 작업과 무관함
- `.codex/INSTALL.md` — 설치 프로세스 변경 없음
- 4가지 옵션 마무리 메뉴 — Claude Code 및 Codex CLI를 위해 그대로 보존됨
- 전체 워크트리 생성 흐름 — 비워크트리 환경을 위해 그대로 보존됨
- 서브에이전트 디스패치/리뷰/반복 루프 — 변경 없음 (파일시스템 공유 확인됨)

## 범위 요약

| 파일 | 변경 내용 |
|---|---|
| `skills/using-git-worktrees/SKILL.md` | +12줄 (Step 0) |
| `skills/finishing-a-development-branch/SKILL.md` | +20줄 (Step 1.5 + 정리 가드) |
| `skills/subagent-driven-development/SKILL.md` | 1줄 수정 |
| `skills/executing-plans/SKILL.md` | 1줄 수정 |
| `skills/using-superpowers/references/codex-tools.md` | +15줄 |

5개 파일에 걸쳐 약 50줄이 추가/변경됩니다. 새로운 파일은 없으며, 기존 기능을 깨뜨리는 변경(breaking change)도 없습니다.

## 향후 고려사항

세 번째 스킬에서도 동일한 감지 패턴이 필요해지면 이를 공유 `references/environment-detection.md` 파일로 추출합니다 (접근법 B). 현재는 2개의 스킬만 사용하므로 필요하지 않습니다.

## 테스트 계획

### 자동화 테스트 (구현 후 Claude Code에서 실행)

1. 일반 저장소 감지 — IN_LINKED_WORKTREE=false 검증
2. 연결된 워크트리 감지 — `git worktree add`로 테스트 워크트리 생성 후 IN_LINKED_WORKTREE=true 검증
3. Detached HEAD 감지 — `git checkout --detach` 후 ON_DETACHED_HEAD=true 검증
4. 마무리 스킬 핸드오프 출력 — 제한된 환경에서 (4가지 옵션 메뉴가 아닌) 핸드오프 메시지가 출력되는지 검증
5. **Step 5 정리 가드** — 연결된 워크트리를 생성(`git worktree add /tmp/test-cleanup -b test-cleanup`)하고 그 안으로 `cd`한 뒤, Step 5 정리 감지(`GIT_DIR` 대 `GIT_COMMON`)를 실행하여 `git worktree remove`를 호출하지 **않음**을 검증합니다. 그런 다음 메인 저장소로 다시 `cd`하여 동일한 감지를 실행하고 `git worktree remove`를 호출**함**을 검증합니다. 이후 테스트 워크트리를 정리합니다.

### 수동 Codex App 테스트 (5개 테스트)

1. Worktree 스레드(workspace-write)에서 감지 — GIT_DIR != GIT_COMMON 및 빈 브랜치 확인
2. Worktree 스레드(Full access)에서 감지 — 동일한 감지, 다른 샌드박스 동작 확인
3. 마무리 스킬 핸드오프 포맷 — 에이전트가 4가지 옵션 메뉴 대신 핸드오프 페이로드를 출력하는지 확인
4. 전체 수명주기 — 감지 → 커밋 → 마무리 감지 → 올바른 동작 → 정리
5. **Local 스레드에서의 샌드박스 폴백** — Codex App **Local 스레드**(workspace-write 샌드박스)를 시작합니다. 프롬프트: "Use the superpowers skill `using-git-worktrees` to set up an isolated workspace for implementing a small change." 사전 확인: `git checkout -b test-sandbox-check`가 `Operation not permitted`로 실패해야 합니다. 예상 결과: 스킬이 `GIT_DIR == GIT_COMMON`(일반 저장소)을 감지하고 `git worktree add -b`를 시도하다가 Seatbelt 거부에 부딪힌 뒤, Step 0의 "이미 작업 공간에 있음" 동작으로 폴백합니다 — 현재 디렉터리에서 설정과 베이스라인 테스트를 실행하고 준비 완료를 보고합니다. 통과 기준: 에이전트가 알 수 없는 에러 메시지 없이 우아하게 복구됨. 실패 기준: 에이전트가 원시 Seatbelt 에러를 출력하거나 재시도하거나 혼란스러운 출력과 함께 포기함.

### 회귀 테스트 (Regression)

- 기존 Claude Code 스킬 트리거링 테스트가 여전히 통과함
- 기존 subagent-driven-development 통합 테스트가 여전히 통과함
- 일반 Claude Code 세션: 전체 워크트리 생성 + 4가지 옵션 마무리가 여전히 작동함
