# 워크트리 대개편: 감지 후 위임 (Worktree Rototill: Detect-and-Defer)

**날짜:** 2026-04-06
**상태:** 초안 (Draft)
**티켓:** PRI-974
**통합(Subsumes):** PRI-823 (Codex App 호환성)

## 문제점

Superpowers는 특정 경로(`.worktrees/<branch>`), 특정 명령어(`git worktree add`), 특정 정리 방식(`git worktree remove`) 등 워크트리 관리에 대해 완고한 방식(opinionated)을 취하고 있습니다. 반면 Claude Code, Codex App, Gemini CLI, Cursor는 모두 자체적인 경로, 수명주기 관리 및 정리 기능을 갖춘 네이티브 워크트리 지원을 제공합니다.

이로 인해 세 가지 실패 모드가 발생합니다:

1. **중복 (Duplication)** — Claude Code에서 스킬이 `EnterWorktree`/`ExitWorktree`가 이미 수행하는 작업을 중복해서 수행합니다.
2. **충돌 (Conflict)** — Codex App에서 스킬이 이미 관리되고 있는 워크트리 내부에 또 다른 워크트리를 생성하려고 시도합니다.
3. **유령 상태 (Phantom state)** — 스킬이 `.worktrees/`에 생성한 워크트리는 하네스(harness)에게 보이지 않고, 하네스가 `.claude/worktrees/`에 생성한 워크트리는 스킬에게 보이지 않습니다.

네이티브 지원이 없는 하네스(Codex CLI, OpenCode, 단독 실행형 Copilot)의 경우 superpowers가 실제 공백을 메워줍니다. 따라서 이 스킬이 사라져서는 안 되며, 네이티브 지원이 존재할 때 방해가 되지 않도록 물러나 있어야 합니다.

## 목표

1. 네이티브 하네스 워크트리 시스템이 존재할 때는 해당 시스템에 위임(defer)합니다.
2. 네이티브 기능이 없는 하네스에 대해서는 워크트리 지원을 계속 제공합니다.
3. `finishing-a-development-branch`의 알려진 버그 세 가지(#940, #999, #238)를 수정합니다.
4. 워크트리 생성을 필수가 아닌 선택 사항(opt-in)으로 만듭니다 (#991).
5. 하드코딩된 `CLAUDE.md` 참조를 플랫폼 중립적인 표현으로 교체합니다 (#1049).

## 비목표 (Non-Goals)

- 워크트리별 환경 규칙 (`.worktree-env.sh`, 포트 오프셋 지정) — Phase 4
- 경로 강제를 위한 PreToolUse 훅 — Phase 4
- 다중 저장소(Multi-repo) 워크트리 문서화 — Phase 4
- 워크트리를 위한 브레인스토밍 체크리스트 변경 — Phase 4
- `.superpowers-session.json` 메타데이터 추적 (흥미로운 PR #997 아이디어지만 v1에는 필요하지 않음)
- 워크트리로의 훅 심볼릭 링크 연결 (PR #965 아이디어, 별개의 관심사)

## 설계 원칙

### 플랫폼이 아닌 상태를 감지하라 (Detect state, not platform)

하네스를 식별하기 위해 환경 변수를 기웃거리는 대신, `GIT_DIR != GIT_COMMON`을 사용하여 "내가 이미 워크트리 안에 있는가?"를 판단합니다. 이는 안정적인 git 프리미티브(2015년 git 2.5부터 도입)로 모든 하네스에서 보편적으로 작동하며, 새로운 하네스가 등장하더라도 유지보수가 전혀 필요 없습니다.

### 선언적 의도, 처방적 폴백 (Declarative intent, prescriptive fallback)

스킬은 목표("작업이 격리된 작업 공간에서 이루어지도록 보장")를 설명하고 사용 가능한 경우 네이티브 도구에 위임합니다. 네이티브 워크트리 지원이 없는 하네스를 위한 폴백으로서만 구체적인 git 명령어를 처방합니다. Step 1a가 먼저 나오며 네이티브 도구들(`EnterWorktree`, `WorktreeCreate`, `/worktree`, `--worktree`)을 명시적으로 지목하고, Step 1b가 git 폴백과 함께 두 번째로 나옵니다. 초기 스펙에서는 Step 1a를 추상적으로("당신은 자신의 툴킷을 알고 있습니다") 유지했으나, TDD를 통해 Step 1a가 너무 모호하면 에이전트가 Step 1b의 구체적인 명령어에 꽂혀버린다(anchoring)는 사실이 입증되었습니다. 네이티브 도구 선호가 안정적으로 작동하게 하려면 명시적인 도구 이름 지목과 동의-권한 부여 브리지(consent-authorization bridge)가 필요했습니다.

### 출처 기반 소유권 (Provenance-based ownership)

워크트리를 생성한 주체가 그 정리(cleanup)에 대한 소유권을 가집니다. 하네스가 생성했다면 superpowers는 이를 건드리지 않습니다. superpowers가 (git 폴백을 통해) 생성했다면 superpowers가 정리합니다. 휴리스틱 기준은 다음과 같습니다: 워크트리가 `.worktrees/` 또는 `worktrees/` 아래에 있으면 superpowers가 소유합니다. 그 외의 모든 경로(`.claude/worktrees/`, `~/.codex/worktrees/`, `.gemini/worktrees/` 또는 구버전의 사용자 글로벌 Superpowers 경로)는 하네스나 사용자의 소유이므로 그대로 둡니다.

## 설계

### 1. `using-git-worktrees` SKILL.md 재작성

이 스킬은 생성 전에 세 가지 새로운 단계를 추가하고 생성 흐름을 단순화합니다.

#### Step 0: 기존 격리 환경 감지 (Detect Existing Isolation)

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

세 가지 결과가 나올 수 있습니다:

| 조건 | 의미 | 동작 |
|-----------|---------|--------|
| `GIT_DIR == GIT_COMMON` | 일반 저장소 체크아웃 | Step 0.5로 진행 |
| `GIT_DIR != GIT_COMMON`, 명명된 브랜치 | 이미 연결된 워크트리 내부에 있음 | Step 3(프로젝트 설정)로 건너뜀. 보고: "Already in isolated workspace at `<path>` on branch `<name>`." |
| `GIT_DIR != GIT_COMMON`, detached HEAD | 외부에서 관리되는 워크트리 (예: Codex App 샌드박스) | Step 3로 건너뜀. 보고: "Already in isolated workspace at `<path>` (detached HEAD, externally managed)." |

Step 0는 누가 워크트리를 생성했는지, 어떤 하네스가 실행 중인지 신경 쓰지 않습니다. 출처와 상관없이 워크트리는 워크트리입니다.

**서브모듈 가드 (Submodule guard):** `GIT_DIR != GIT_COMMON`은 git 서브모듈 내부에서도 참(true)이 됩니다. "이미 워크트리에 있다"고 결론 내리기 전에 현재 위치가 서브모듈이 아닌지 확인합니다:

```bash
# 이 명령어가 경로를 반환하면 워크트리가 아니라 서브모듈 내부에 있는 것임
git rev-parse --show-superproject-working-tree 2>/dev/null
```

서브모듈 내부에 있다면 `GIT_DIR == GIT_COMMON`으로 간주합니다 (Step 0.5로 진행).

#### Step 0.5: 동의 구하기 (Consent)

Step 0에서 기존 격리 환경을 찾지 못한 경우(`GIT_DIR == GIT_COMMON`), 생성하기 전에 먼저 물어봅니다:

> "Would you like me to set up an isolated worktree? This protects your current branch from changes. (y/n)"

예(yes)인 경우 Step 1로 진행합니다. 아니요(no)인 경우 현재 위치에서 작업합니다 — 워크트리 없이 Step 3로 건너뜁니다.

Step 0에서 기존 격리 환경이 감지되면 이 단계는 완전히 건너뜁니다 (이미 존재하는 것에 대해 물어볼 필요가 없으므로).

#### Step 1a: 네이티브 도구 (권장)

> 사용자가 격리된 작업 공간을 요청했습니다 (Step 0 동의). 사용 가능한 도구를 확인하세요 — `EnterWorktree`, `WorktreeCreate`, `/worktree` 명령어 또는 `--worktree` 플래그가 있나요? 예(YES)라면: 워크트리를 생성하겠다는 사용자의 동의가 곧 해당 도구를 사용할 수 있는 권한 부여(authorization)입니다. 지금 바로 해당 도구를 사용하고 Step 3로 건너뛰세요.

네이티브 도구를 사용한 후에는 Step 3(프로젝트 설정)로 건너뜁니다.

**설계 노트 — TDD 수정 사항:** 초기 스펙에서는 의도적으로 짧고 추상적인 Step 1a("당신은 자신의 툴킷을 알고 있으므로 스킬이 특정 도구를 지목할 필요는 없습니다")를 사용했습니다. 그러나 TDD 검증 결과 이는 틀렸음이 입증되었습니다: 에이전트들은 Step 1b의 구체적인 git 명령어에 꽂혀 추상적인 지침을 무시했습니다 (통과율 2/6). 세 가지 변경을 통해 이를 해결했습니다 (GREEN 및 PRESSURE 테스트에 걸쳐 50/50 통과율 달성):

1. **명시적 도구 지목** — `EnterWorktree`, `WorktreeCreate`, `/worktree`, `--worktree`의 이름을 직접 나열함으로써 결정을 해석의 영역("내게 네이티브 도구가 있는가?")에서 사실 조회의 영역("내 도구 목록에 `EnterWorktree`가 있는가?")으로 전환합니다. 이러한 도구가 없는 플랫폼의 에이전트는 단순히 목록을 확인하고 아무것도 찾지 못한 뒤 자연스럽게 Step 1b로 넘어갑니다. 오탐(false positive)은 전혀 관찰되지 않았습니다.
2. **동의 브리지 (Consent bridge)** — "워크트리를 생성하겠다는 사용자의 동의가 곧 해당 도구를 사용할 수 있는 권한 부여입니다"라는 문구는 `EnterWorktree`의 도구 수준 가드레일("사용자가 명시적으로 요청할 때만(ONLY when user explicitly asks)")을 정면으로 해결합니다. 도구 설명은 스킬 지침보다 우선하므로(Claude Code #29950), 스킬은 사용자의 동의를 도구가 요구하는 권한 부여로 프레이밍해야 합니다.
3. **Red Flag 항목** — Red Flags 섹션에 구체적인 안티패턴("네이티브 워크트리 도구가 있는데도 `git worktree add`를 사용하는 것 — 이것이 가장 흔한 실수(#1 mistake)입니다")을 명시합니다.

파일 분리(Step 1b를 별도의 스킬로 분리하는 것)도 테스트해 보았으나 불필요한 것으로 입증되었습니다. 앵커링(anchoring) 문제는 git 명령어의 물리적 분리가 아니라 Step 1a 텍스트의 품질로 해결됩니다. 전체 240줄짜리 스킬(모든 git 명령어가 노출됨)로 진행한 대조군 테스트에서 20/20을 통과했습니다.

#### Step 1b: Git 워크트리 폴백 (Git Worktree Fallback)

사용 가능한 네이티브 도구가 없을 때 수동으로 워크트리를 생성합니다.

**디렉터리 선택** (우선순위 순):
1. 프로젝트의 에이전트 지침 파일(CLAUDE.md, GEMINI.md, AGENTS.md, .cursorrules 또는 이에 준하는 파일)에서 선호하는 워크트리 디렉터리 설정이 있는지 확인합니다.
2. 기존 `.worktrees/` 또는 `worktrees/` 디렉터리가 있는지 확인합니다 — 발견되면 해당 디렉터리를 사용합니다. 둘 다 존재하면 `.worktrees/`가 우선합니다.
3. 기본값으로 `.worktrees/`를 사용합니다.

대화형 디렉터리 선택 프롬프트는 없습니다. 구버전의 사용자 글로벌 Superpowers 워크트리 경로는 감지하거나 제안하지 않으며, 새로운 수동 워크트리는 사용자가 명시적으로 다른 위치를 지정하지 않는 한 프로젝트 로컬에 생성됩니다.

**안전성 검증** (프로젝트 로컬 디렉터리만 해당):

```bash
git check-ignore -q .worktrees 2>/dev/null
```

무시(ignore) 처리되어 있지 않다면 `.gitignore`에 추가하고 커밋한 뒤 진행합니다.

**생성:**

```bash
git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

**훅 인식 (Hooks awareness):** Git 워크트리는 부모 저장소의 hooks 디렉터리를 상속하지 않습니다. 1b를 통해 워크트리를 생성한 후, 메인 저장소에 hooks 디렉터리가 존재하면 심볼릭 링크로 연결합니다:

```bash
if [ -d "$MAIN_ROOT/.git/hooks" ]; then
    ln -sf "$MAIN_ROOT/.git/hooks" "$path/.git/hooks"
fi
```

이렇게 하면 작업이 워크트리로 이동했을 때 pre-commit 검사, 린터 및 기타 훅이 소리 없이 작동을 멈추는 것을 방지할 수 있습니다. (PR #965에서 착안한 아이디어.)

**샌드박스 폴백:** `git worktree add`가 권한 에러로 실패하면 제한된 환경으로 간주합니다. 생성을 건너뛰고 현재 디렉터리에서 작업하며 Step 3로 진행합니다.

**단계 번호 매기기 참고:** 현재 스킬은 Step 1~4를 평면적인 목록으로 가지고 있습니다. 이 재설계는 0, 0.5, 1a, 1b, 3, 4를 사용합니다. Step 2는 없습니다 — 기존에는 단일 구조의 "Create Isolated Workspace"였으나 이제 1a/1b 구조로 분할되었습니다. 구현 시에는 번호를 깔끔하게 다시 매기거나(예: 0 → "Step 0: Detect", 0.5 → Step 0의 흐름 내부에 포함, 1a/1b → "Step 1", 3 → "Step 2", 4 → "Step 3") 참고 문구와 함께 현재 번호를 유지할 수 있습니다. 구현자의 선택에 맡깁니다.

#### Step 3-4: 프로젝트 설정 및 베이스라인 테스트 (변경 없음)

어떤 경로로 작업 공간이 준비되었든(Step 0에서 기존 환경 감지, Step 1a 네이티브 도구, Step 1b git 폴백, 또는 워크트리 전혀 없음), 이후 실행 흐름은 하나로 수렴합니다:

- **Step 3:** 프로젝트 설정을 자동으로 감지하고 실행합니다 (`npm install`, `cargo build`, `pip install`, `go mod download` 등).
- **Step 4:** 테스트 스위트를 실행합니다. 테스트가 실패하면 실패 내역을 보고하고 계속 진행할지 물어봅니다.

### 2. `finishing-a-development-branch` SKILL.md 재작성

마무리 스킬에 환경 감지 기능이 추가되고 세 가지 버그가 수정됩니다.

#### Step 1: 테스트 검증 (변경 없음)

프로젝트의 테스트 스위트를 실행합니다. 테스트가 실패하면 멈춥니다. 완료 옵션을 제시하지 마세요.

#### Step 1.5: 환경 감지 (신규)

생성 시의 Step 0와 동일한 감지를 다시 실행합니다:

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
```

세 가지 경로가 있습니다:

| 상태 | 메뉴 | 정리 (Cleanup) |
|-------|------|---------|
| `GIT_DIR == GIT_COMMON` (일반 저장소) | 표준 4가지 옵션 | 정리할 워크트리 없음 |
| `GIT_DIR != GIT_COMMON`, 명명된 브랜치 | 표준 4가지 옵션 | 출처 기반 (Step 5 참조) |
| `GIT_DIR != GIT_COMMON`, detached HEAD | 축소된 메뉴: 새 브랜치로 푸시 후 PR 생성, 현재 상태 유지, 폐기 | 병합 옵션 없음 (detached HEAD에서는 병합 불가) |

#### Step 2: 베이스 브랜치 결정 (변경 없음)

#### Step 3: 옵션 제시

**일반 저장소 및 명명된 브랜치 워크트리:**

1. 로컬에서 `<base-branch>`에 다시 병합 (Merge back to `<base-branch>` locally)
2. 푸시 및 Pull Request 생성 (Push and create a Pull Request)
3. 브랜치를 현재 상태로 유지 - 나중에 직접 처리 (Keep the branch as-is (I'll handle it later))
4. 이 작업 폐기 (Discard this work)

**Detached HEAD:**

1. 새 브랜치로 푸시 및 Pull Request 생성 (Push as new branch and create a Pull Request)
2. 현재 상태로 유지 - 나중에 직접 처리 (Keep as-is (I'll handle it later))
3. 이 작업 폐기 (Discard this work)

#### Step 4: 선택 실행

**옵션 1 (로컬 병합):**

```bash
# CWD 안전성을 위해 메인 저장소 루트 확보 (버그 #238 수정)
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"

# 무언가를 제거하기 전에 먼저 병합하고 성공 여부를 검증함
git checkout <base-branch>
git pull
git merge <feature-branch>
<run tests>

# 병합이 성공한 후에만: 워크트리를 제거한 다음 브랜치를 삭제함 (버그 #999 수정)
git worktree remove "$WORKTREE_PATH"  # superpowers가 소유한 경우에만
git branch -d <feature-branch>
```

순서가 매우 중요합니다: 병합 → 검증 → 워크트리 제거 → 브랜치 삭제. 기존 스킬은 워크트리를 제거하기 전에 브랜치를 삭제했습니다 (워크트리가 여전히 브랜치를 참조하고 있기 때문에 실패함). 워크트리를 먼저 제거하는 단순한 수정 방식도 잘못되었습니다 — 만약 그 후 병합이 실패하면 작업 디렉터리가 사라져 변경 사항을 잃게 되기 때문입니다.

**옵션 2 (PR 생성):**

브랜치를 푸시하고 PR을 생성합니다. 워크트리는 정리하지 **마세요** — 사용자가 PR 반복 작업을 위해 필요로 합니다. (버그 #940 수정: 모순되는 "Then: Cleanup worktree" 문구 제거.)

**옵션 3 (현재 상태 유지):** 아무 작업도 하지 않습니다.

**옵션 4 (폐기):** "discard"를 직접 타이핑하여 확인받도록 요구합니다. 그런 다음 (superpowers가 소유한 경우) 워크트리를 제거하고 브랜치를 강제 삭제합니다.

#### Step 5: 정리 (업데이트됨)

```
if GIT_DIR == GIT_COMMON:
    # 일반 저장소, 정리할 워크트리 없음
    done

if worktree path is under .worktrees/ or worktrees/:
    # Superpowers가 생성함 — 우리가 정리 소유권을 가짐
    cd to main repo root       # 버그 #238 수정
    git worktree remove <path>

else:
    # 하네스가 생성함 — 손대지 않음(hands off)
    # 플랫폼이 작업 공간 종료(workspace-exit) 도구를 제공한다면 그것을 사용함
    # 그렇지 않다면 워크트리를 그대로 둠
```

정리는 옵션 1과 4에 대해서만 실행됩니다. 옵션 2와 3은 항상 워크트리를 보존합니다. (버그 #940 수정.)

**오래된 워크트리 가지치기 (Stale worktree pruning):** `git worktree remove`를 실행한 후에는 항상 자가 치유(self-healing) 단계로 `git worktree prune`을 실행합니다. 워크트리 디렉터리는 대역 외(out-of-band)에서 삭제될 수 있으며(예: 하네스 정리, 수동 `rm`, `.claude/` 정리 등), 이로 인해 오래된 등록 정보가 남아 혼란스러운 에러를 유발할 수 있습니다. 단 한 줄로 소리 없는 부패(silent rot)를 방지합니다. (PR #1072에서 착안한 아이디어.)

### 3. 통합(Integration) 업데이트

#### `subagent-driven-development` 및 `executing-plans`

두 스킬 모두 현재 통합 섹션에서 `using-git-worktrees`를 REQUIRED로 나열하고 있습니다. 이를 다음과 같이 변경합니다:

> `using-git-worktrees` — Ensures isolated workspace (creates one or verifies existing)

이제 스킬 자체가 동의(Step 0.5)와 감지(Step 0)를 처리하므로, 이를 호출하는 스킬들이 별도로 관문을 두거나 프롬프트를 띄울 필요가 없습니다.

#### `writing-plans`

오래된 주장인 "should be run in a dedicated worktree (created by brainstorming skill)" 문구를 제거합니다. 브레인스토밍은 설계 스킬이며 워크트리를 생성하지 않습니다. 워크트리 프롬프트는 실행 시점에 `using-git-worktrees`를 통해 이루어집니다.

### 4. 플랫폼 중립적인 지침 파일 참조

워크트리 관련 스킬에 하드코딩된 모든 `CLAUDE.md` 인스턴스를 다음으로 교체합니다:

> "your project's agent instruction file (CLAUDE.md, GEMINI.md, AGENTS.md, .cursorrules, or equivalent)"

이는 Step 1b의 디렉터리 선호도 확인에 적용됩니다.

## 버그 수정 (포함됨)

| 버그 | 문제점 | 수정 내용 | 위치 |
|-----|---------|-----|----------|
| #940 | 옵션 2 본문에는 "Then: Cleanup worktree (Step 5)"라고 되어 있지만 빠른 참조(quick reference)에는 유지하라고 되어 있음. Step 5에는 "For Options 1, 2, 4"라고 되어 있지만 Common Mistakes에는 "Options 1 and 4 only"라고 되어 있음. | 옵션 2에서 정리 단계 제거. Step 5는 옵션 1과 4에만 적용됨. | finishing SKILL.md |
| #999 | 옵션 1이 워크트리를 제거하기 전에 브랜치를 삭제함. 워크트리가 여전히 브랜치를 참조하고 있어 `git branch -d`가 실패할 수 있음. | 순서를 다음과 같이 재조정: 병합 → 테스트 검증 → 워크트리 제거 → 브랜치 삭제. 무언가를 제거하기 전에 반드시 병합이 먼저 성공해야 함. | finishing SKILL.md |
| #238 | 현재 작업 디렉터리(CWD)가 제거하려는 워크트리 내부에 있으면 `git worktree remove`가 조용히 실패함. | CWD 가드 추가: `git worktree remove` 실행 전에 메인 저장소 루트로 `cd`함. | finishing SKILL.md |

## 해결된 이슈

| 이슈 | 해결 방법 |
|-------|-----------|
| #940 | 직접 수정 (버그 #940) |
| #991 | Step 0.5의 옵트인(opt-in) 동의 |
| #918 | Step 0 감지 + Step 1.5 마무리 감지 |
| #1009 | Step 1a를 통해 해결됨 — 에이전트가 하네스 네이티브 경로에 생성하는 네이티브 도구(예: `EnterWorktree`)를 사용함. Step 1a가 제대로 작동하는지에 달려 있음(위험 요소 참조). |
| #999 | 직접 수정 (버그 #999) |
| #238 | 직접 수정 (버그 #238) |
| #1049 | 플랫폼 중립적인 지침 파일 참조 |
| #279 | 감지 후 위임(detect-and-defer)으로 해결됨 — 네이티브 경로를 덮어쓰지 않으므로 그대로 존중됨 |
| #574 | **보류됨(Deferred).** 이 스펙의 어떤 내용도 버그가 존재하는 브레인스토밍 스킬을 건드리지 않습니다. 전체 수정(브레인스토밍 체크리스트에 워크트리 단계 추가)은 Phase 4에서 다룹니다. |

## 위험 요소 (Risks)

### Step 1a는 핵심 전제 조건임 — 해결됨 (RESOLVED)

Step 1a(에이전트가 git 폴백보다 네이티브 워크트리 도구를 선호하는 것)는 전체 설계가 의존하는 기반입니다. 만약 네이티브 지원이 있는 하네스에서 에이전트가 Step 1a를 무시하고 Step 1b로 넘어가 버리면 감지 후 위임(detect-and-defer)은 완전히 실패합니다.

**상태:** 이 위험은 구현 중에 실제로 현실화되었습니다. 초기의 추상적인 Step 1a("당신은 자신의 툴킷을 알고 있습니다")는 Claude Code에서 2/6의 성공률로 실패했습니다. 하지만 TDD 게이트가 설계된 대로 작동하여 어떤 스킬 파일도 수정되기 전에 실패를 잡아냈고, 깨진 릴리스를 방지했습니다. 세 번의 REFACTOR 반복을 통해 근본 원인(구체적인 명령어에 대한 에이전트 앵커링, 스킬 지침을 덮어쓰는 도구 설명 가드레일)을 파악했고, GREEN 및 PRESSURE 테스트에 걸쳐 50/50으로 검증된 수정안을 만들어냈습니다. 자세한 내용은 위의 Step 1a 설계 노트를 참조하세요.

**크로스 플랫폼 검증:**

2026-04-06 기준으로 Claude Code는 에이전트가 세션 중간에 호출할 수 있는 워크트리 도구(`EnterWorktree`)를 가진 유일한 하네스입니다. 다른 모든 하네스는 에이전트가 시작되기 전에 워크트리를 생성하거나(Codex App, Gemini CLI, Cursor) 네이티브 워크트리 지원이 없습니다(Codex CLI, OpenCode). Step 1a는 상위 호환성(forward-compatible)을 가집니다: 다른 하네스들이 에이전트가 호출 가능한 워크트리 도구를 추가하면, 에이전트는 스킬 변경 없이도 나열된 예시와 대조하여 해당 도구를 사용할 것입니다.

| 하네스 | 현재 워크트리 모델 | 스킬 메커니즘 | 테스트 결과 |
|---------|----------------------|-----------------|--------|
| Claude Code | 에이전트 호출 가능 `EnterWorktree` | Step 1a | 50/50 (GREEN + PRESSURE) |
| Codex CLI | 네이티브 도구 없음 (셸 전용) | Step 1b git 폴백 | 6/6 (`codex exec`) |
| Gemini CLI | 시작 시점 `--worktree` 플래그, 에이전트 도구 없음 | 플래그로 시작된 경우 Step 0, 아닌 경우 Step 1b | Step 0: 1/1, Step 1b: 1/1 (`gemini -p`) |
| Cursor Agent | 사용자 대면 `/worktree`, 에이전트 도구 없음 | 사용자가 활성화한 경우 Step 0, 아닌 경우 Step 1b | Step 0: 1/1, Step 1b: 1/1 (`cursor-agent -p`) |
| Codex App | 플랫폼 관리형, detached HEAD, 에이전트 도구 없음 | Step 0에서 기존 환경 감지 | 1/1 시뮬레이션됨 |
| OpenCode | 감지 전용 (`ctx.worktree`), 에이전트 도구 없음 | Step 1b git 폴백 | 테스트되지 않음 (CLI 접근 불가) |

**잔여 위험:**
1. 만약 Anthropic이 `EnterWorktree`의 도구 설명을 더 제한적으로 변경한다면(예: "스킬 지침에 기반하여 사용하지 마세요"), 동의 브리지가 깨지게 됩니다. 도구 설명이 스킬 기반 호출을 수용하도록 요청하는 이슈를 등록할 가치가 있습니다.
2. 다른 하네스들이 에이전트 호출 가능 워크트리 도구를 추가할 때 Step 1a의 목록에 없는 이름을 사용할 수 있습니다. 새로운 도구가 등장함에 따라 목록을 업데이트해야 합니다. 일반적인 표현("워크트리 또는 작업 공간 격리 도구")이 어느 정도 상위 호환성을 제공하기는 합니다.

### 출처 휴리스틱 (Provenance heuristic)

`.worktrees/` 또는 `worktrees/`는 우리 것, 그 외에는 손대지 않음(hands off)이라는 휴리스틱은 현재의 모든 하네스에서 잘 작동합니다. 만약 미래의 어떤 하네스가 프로젝트 로컬 디렉터리 중 하나를 자체 규칙으로 채택한다면 오탐(superpowers가 하네스 소유 워크트리를 정리하려고 시도함)이 발생할 수 있습니다. 마찬가지로 사용자가 superpowers 없이 수동으로 `git worktree add .worktrees/experiment`를 실행한 경우에도 우리가 잘못해서 소유권을 주장하게 됩니다. 모든 하네스가 브랜드화된 경로를 사용하고 수동으로 `.worktrees/`를 생성할 가능성은 낮으므로 두 경우 모두 위험도는 낮지만, 참고할 가치는 있습니다.

### Detached HEAD 마무리

Detached HEAD 워크트리에 대한 축소된 메뉴(병합 옵션 없음)는 Codex App의 샌드박스 모델에 부합합니다. 사용자가 다른 이유로 detached HEAD 상태에 있더라도 축소된 메뉴는 여전히 합리적입니다 — 실제로 브랜치를 먼저 생성하지 않고서는 detached HEAD에서 병합을 수행할 수 없기 때문입니다.

## 구현 참고사항

두 스킬 파일 모두 핵심 단계 외에 구현 중에 업데이트해야 하는 섹션들을 포함하고 있습니다:

- **프론트매터 (Frontmatter)** (`name`, `description`): 감지 후 위임(detect-and-defer) 동작을 반영하도록 업데이트
- **빠른 참조 (Quick Reference) 표**: 새로운 단계 구조와 버그 수정에 맞춰 재작성
- **흔한 실수 (Common Mistakes) 섹션**: 이전 동작을 참조하는 항목 업데이트 또는 제거 (예: "Skip CLAUDE.md check"는 이제 틀린 내용임)
- **Red Flags 섹션**: 새로운 우선순위를 반영하도록 업데이트 (예: "Step 0에서 기존 격리 환경이 감지되었을 때 절대 워크트리를 생성하지 마세요")
- **통합 (Integration) 섹션**: 스킬 간의 상호 참조 업데이트

이 스펙은 *무엇이 변경되는지*를 설명하며, 이러한 부수적 섹션들에 대한 정확한 수정 사항은 구현 계획에서 지정할 것입니다.

## 향후 작업 (이 스펙에 포함되지 않음)

- **Phase 3 잔여 작업:** `$TMPDIR` 디렉터리 옵션 (#666), 캐싱 및 환경 상속에 대한 설정 문서 (#299)
- **Phase 4:** 경로 강제를 위한 PreToolUse 훅 (#1040), 워크트리별 환경 규칙 (#597), 브레인스토밍 체크리스트 워크트리 단계 (#574), 다중 저장소 문서화 (#710)
