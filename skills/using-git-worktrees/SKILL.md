---
name: using-git-worktrees
description: 현재 작업 공간(workspace)으로부터 격리가 필요한 기능 작업을 시작할 때 또는 구현 계획(implementation plan)을 실행하기 전에 사용 — 네이티브 도구나 git worktree 폴백(fallback)을 통해 격리된 작업 공간이 존재하도록 보장합니다.
---

# Git 워크트리 사용하기 (Using Git Worktrees)

## 개요 (Overview)

작업이 격리된 작업 공간(isolated workspace)에서 이루어지도록 보장합니다. 플랫폼의 네이티브 워크트리(worktree) 도구를 우선적으로 사용하세요. 사용할 수 있는 네이티브 도구가 없을 때만 수동 `git worktree` 방식으로 폴백(fallback)합니다.

**핵심 원칙 (Core principle):** 먼저 기존의 격리 환경을 감지합니다. 그다음 네이티브 도구를 사용합니다. 그 후에만 git으로 폴백합니다. 절대 하네스(harness)와 맞서 싸우지 마세요.

**시작 시 선언 (Announce at start):** "`using-git-worktrees` 스킬을 사용하여 격리된 작업 공간을 설정하겠습니다." ("I'm using the using-git-worktrees skill to set up an isolated workspace.")

## 0단계: 기존 격리 환경 감지 (Step 0: Detect Existing Isolation)

**무언가를 생성하기 전에, 이미 격리된 작업 공간 안에 있는지 확인하세요.**

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

**서브모듈 가드 (Submodule guard):** `GIT_DIR != GIT_COMMON` 조건은 git 서브모듈(submodule) 내부에서도 참(true)이 됩니다. "이미 워크트리 안에 있음"이라고 결론 내리기 전에, 서브모듈 내부가 아닌지 검증하세요:

```bash
# 이 명령이 경로를 반환하면 워크트리가 아니라 서브모듈 내부에 있는 것입니다 — 일반 저장소로 취급하세요
git rev-parse --show-superproject-working-tree 2>/dev/null
```

**`GIT_DIR != GIT_COMMON`인 경우 (그리고 서브모듈이 아닌 경우):** 이미 연결된 워크트리(linked worktree) 내부에 있습니다. 2단계(프로젝트 설정)로 건너뛰세요. 또 다른 워크트리를 생성하지 마세요.

브랜치 상태와 함께 보고합니다:
- 브랜치 위에 있는 경우: "이미 `<path>` 경로의 `<name>` 브랜치에 있는 격리된 작업 공간에 있습니다." ("Already in isolated workspace at `<path>` on branch `<name>`.")
- 분리된 HEAD(Detached HEAD)인 경우: "이미 `<path>` 경로의 격리된 작업 공간에 있습니다(분리된 HEAD, 외부에서 관리됨). 작업 완료 시점에 브랜치 생성이 필요합니다." ("Already in isolated workspace at `<path>` (detached HEAD, externally managed). Branch creation needed at finish time.")

**`GIT_DIR == GIT_COMMON`인 경우 (또는 서브모듈 내부인 경우):** 일반 저장소 체크아웃(checkout) 상태에 있습니다.

사용자가 지침(instructions)에서 이미 워크트리 선호 사항을 밝혔나요? 그렇지 않다면 워크트리를 생성하기 전에 동의를 구하세요:

> "격리된 워크트리를 설정해 드릴까요? 현재 브랜치가 변경 사항으로부터 보호됩니다." ("Would you like me to set up an isolated worktree? It protects your current branch from changes.")

이미 선언된 선호 사항이 있다면 묻지 않고 그대로 따릅니다. 사용자가 동의하지 않으면 현재 위치에서 작업하고 2단계로 건너뛰세요.

## 1단계: 격리된 작업 공간 생성 (Step 1: Create Isolated Workspace)

**두 가지 메커니즘이 있습니다. 다음 순서대로 시도하세요.**

### 1a. 네이티브 워크트리 도구 (권장) (Native Worktree Tools (preferred))

사용자가 격리된 작업 공간을 요청했습니다(0단계 동의). 워크트리를 생성할 수 있는 도구가 이미 있나요? `EnterWorktree`, `WorktreeCreate` 같은 이름의 도구이거나, `/worktree` 명령어, 혹은 `--worktree` 플래그일 수 있습니다. 있다면 그것을 사용하고 2단계로 건너뛰세요.

네이티브 도구는 디렉토리 배치, 브랜치 생성, 정리를 자동으로 처리합니다. 네이티브 도구가 있는데도 `git worktree add`를 사용하면 하네스가 인식하거나 관리할 수 없는 유령 상태(phantom state)가 만들어집니다.

사용 가능한 네이티브 워크트리 도구가 전혀 없을 때만 1b단계로 진행하세요.

### 1b. Git 워크트리 폴백 (Git Worktree Fallback)

**1a단계가 해당하지 않을 때만 이 방법을 사용하세요** — 즉, 사용할 수 있는 네이티브 워크트리 도구가 없을 때입니다. git을 사용해 수동으로 워크트리를 생성합니다.

#### 디렉토리 선택 (Directory Selection)

다음 우선순위를 따르세요. 명시적인 사용자 선호 사항은 관찰된 파일시스템 상태보다 항상 우선합니다.

1. **지침에서 선언된 워크트리 디렉토리 선호 사항이 있는지 확인합니다.** 사용자가 이미 지정했다면 묻지 않고 그 위치를 사용하세요.

2. **기존의 프로젝트 로컬 워크트리 디렉토리가 있는지 확인합니다:**
   ```bash
   ls -d .worktrees 2>/dev/null     # 권장 (숨김 디렉토리)
   ls -d worktrees 2>/dev/null      # 대안
   ```
   발견되면 해당 디렉토리를 사용합니다. 둘 다 존재하면 `.worktrees`가 우선합니다.

3. **다른 지침이 전혀 없다면**, 프로젝트 루트의 `.worktrees/`를 기본값으로 사용합니다.

#### 안전성 검증 (프로젝트 로컬 디렉토리만 해당) (Safety Verification (project-local directories only))

**워크트리를 생성하기 전에 해당 디렉토리가 git에서 무시(ignore) 처리되어 있는지 반드시 검증해야 합니다:**

```bash
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

**무시 처리되어 있지 않다면(If NOT ignored):** `.gitignore`에 추가하고 변경 사항을 커밋한 뒤 진행하세요.

**중요한 이유:** 워크트리 내용물이 실수로 저장소에 커밋되는 것을 방지합니다.

#### 워크트리 생성 (Create the Worktree)

```bash
# 선택된 위치를 기반으로 경로 결정
path="$LOCATION/$BRANCH_NAME"

git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

**샌드박스 폴백 (Sandbox fallback):** 만약 `git worktree add`가 권한 오류(샌드박스 거부)로 실패하면, 사용자에게 샌드박스가 워크트리 생성을 차단하여 현재 디렉토리에서 대신 작업한다고 알리세요. 그런 다음 현재 위치에서 설정 및 베이스라인 테스트(baseline tests)를 실행합니다.

## 2단계: 프로젝트 설정 (Step 2: Project Setup)

프로젝트에 맞는 설정을 자동으로 감지하고 실행합니다:

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

## 3단계: 깨끗한 베이스라인 검증 (Step 3: Verify Clean Baseline)

작업 공간이 깨끗한 상태에서 시작하는지 확인하기 위해 테스트를 실행합니다:

```bash
# 프로젝트에 적합한 명령어 사용
npm test / cargo test / pytest / go test ./...
```

**테스트가 실패하는 경우:** 실패 내역을 보고하고, 그대로 진행할지 아니면 원인을 조사할지 질문하세요.

**테스트가 통과하는 경우:** 준비 완료를 보고합니다.

### 보고 (Report)

```
Worktree ready at <full-path>
Tests passing (<N> tests, 0 failures)
Ready to implement <feature-name>
```

(한국어 의미 참고: `<full-path>`에 워크트리 준비 완료 / 테스트 통과 (`<N>`개 테스트, 실패 0개) / `<feature-name>` 구현 준비 완료)

## 빠른 참조 (Quick Reference)

| 상황 (Situation) | 조치 (Action) |
|-----------|--------|
| 이미 연결된 워크트리 내부에 있음 | 생성 건너뛰기 (0단계) |
| 서브모듈 내부에 있음 | 일반 저장소로 취급 (0단계 가드) |
| 네이티브 워크트리 도구 사용 가능 | 해당 도구 사용 (1a단계) |
| 네이티브 도구 없음 | Git 워크트리 폴백 (1b단계) |
| `.worktrees/` 존재함 | 해당 디렉토리 사용 (ignore 여부 검증) |
| `worktrees/` 존재함 | 해당 디렉토리 사용 (ignore 여부 검증) |
| 둘 다 존재함 | `.worktrees/` 사용 |
| 둘 다 존재하지 않음 | 지침 파일 확인 후, 기본값 `.worktrees/` 사용 |
| 디렉토리가 ignore 처리되지 않음 | `.gitignore`에 추가 + 커밋 |
| 생성 시 권한 오류 발생 | 샌드박스 폴백, 현재 위치에서 작업 |
| 베이스라인 검증 중 테스트 실패 | 실패 보고 + 질문 |
| `package.json`/`Cargo.toml` 없음 | 의존성 설치 건너뛰기 |

## 흔한 합리화 (Common Rationalizations)

| 변명 (Excuse) | 현실 (Reality) |
|--------|---------|
| "보나 마나 워크트리 안이 아니니 확인할 필요 없어" | 0단계를 실행하세요. 하네스가 생성한 격리 환경과 서브모듈 모두 눈대중만으로는 속기 쉽습니다. 감지 명령어가 확실히 결론을 내줍니다. |
| "네이티브 도구를 찾는 것보다 `git worktree add`가 더 빨라" | 네이티브 도구(예: `EnterWorktree`)는 디렉토리 배치, 브랜치 생성, 정리를 직접 담당합니다. 이를 우회하는 것이 가장 흔한 1순위 실수이며, 하네스가 볼 수도 관리할 수도 없는 유령 상태(phantom state)를 만듭니다. |
| "워크트리 디렉토리는 당연히 이미 ignore 되어 있겠지" | `git check-ignore`를 실행하세요. ignore 처리되지 않은 워크트리 디렉토리는 트리 전체를 저장소에 커밋해 버립니다. |
| "디렉토리 이름이야 아무거나 써도 되지" | 명시적 지침이 기존의 프로젝트 로컬 디렉토리보다 우선하며, 기존 프로젝트 로컬 디렉토리는 `.worktrees/` 기본값보다 우선합니다. |
| "방금 만든 깨끗한 작업 공간이니 베이스라인 테스트는 나중에 해도 돼" | 베이스라인이 오염되어 있으면 이후 발생하는 모든 실패의 원인이 모호해집니다. 지금 바로 테스트를 실행하세요. 실패를 무시하고 진행할지 여부는 인간 파트너(human partner)가 결정할 몫입니다. |
