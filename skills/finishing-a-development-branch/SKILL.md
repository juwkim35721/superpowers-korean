---
name: finishing-a-development-branch
description: 구현이 완료되고 모든 테스트가 통과하여 작업 결과를 어떻게 통합(integrate)할지 결정해야 할 때 사용
---

# 개발 브랜치 마무리하기 (Finishing a Development Branch)

## 개요 (Overview)

**핵심 원칙 (Core principle):** 테스트 검증 → 환경 감지 → 옵션 제시 → 선택 실행 → 정리(Clean up).

**시작 시 선언 (Announce at start):** "`finishing-a-development-branch` 스킬을 사용하여 이 작업을 완료하겠습니다." ("I'm using the finishing-a-development-branch skill to complete this work.")

## 1단계: 테스트 검증 (Step 1: Verify Tests)

프로젝트의 전체 테스트 스위트를 실행합니다 (`npm test` / `cargo test` / `pytest` / `go test ./...`).

**테스트가 실패하는 경우**, 실패 내역을 보고하고 멈추세요 — 메뉴는 모든 테스트가 통과(green suite)한 후에야 제시할 수 있습니다:

```
Tests failing (<N> failures). Must fix before completing:

[Show failures]
```

**테스트가 통과하는 경우:** 2단계로 계속 진행합니다.

## 2단계: 환경 감지 (Step 2: Detect Environment)

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
# 정리 단계(6단계)에서 이 값이 필요하지만 5단계에서 디렉토리를 변경하므로,
# 아직 작업 공간(workspace) 내부에 있을 때 지금 캡처해 둡니다
WORKTREE_PATH=$(git rev-parse --show-toplevel)
```

이 결과에 따라 어떤 메뉴를 보여줄지와 정리(cleanup)를 어떻게 수행할지가 결정됩니다:

| 상태 (State) | 메뉴 (Menu) | 정리 (Cleanup) |
|-------|------|---------|
| `GIT_DIR == GIT_COMMON` (일반 저장소) | 표준 3가지 옵션 | 정리할 워크트리 없음 |
| `GIT_DIR != GIT_COMMON`, 이름 있는 브랜치 (named branch) | 표준 3가지 옵션 | 생성 출처(provenance) 기반 정리 (6단계 참조) |
| `GIT_DIR != GIT_COMMON`, 분리된 HEAD (detached HEAD) | 축소된 2가지 옵션 (병합 없음) | 외부에서 관리됨 — 그대로 유지 |

## 3단계: 베이스 브랜치 확인 (Step 3: Determine Base Branch)

베이스 브랜치(base branch)는 현재 작업이 분기되어 나온 브랜치로, 보통 계획(plan), 대화 내용, 또는 브랜치의 업스트림(upstream)에 명시되어 있습니다. 아직 확실히 알지 못한다면 다음과 같이 물어보세요: "이 브랜치는 `<추측한 브랜치명>`에서 분기된 것으로 보이는데, 맞나요?" ("This branch split from `<your best guess>` - is that correct?")
병합하기 전에 반드시 확인하세요. 잘못된 베이스 브랜치에 병합하면 되돌리는 데 큰 비용이 듭니다.

## 4단계: 옵션 제시 (Step 4: Present Options)

**일반 저장소 및 이름 있는 브랜치 워크트리 — 정확히 다음 3가지 옵션을 제시하세요:**

```
구현이 완료되었습니다. 어떻게 처리할까요? (Implementation complete. What would you like to do?)

1. 로컬에서 <base-branch>에 병합 (Merge back to <base-branch> locally)
2. 푸시 후 Pull Request 생성 (Push and create a Pull Request)
3. 브랜치를 현재 상태로 유지 (나중에 직접 처리) (Keep the branch as-is (I'll handle it later))

어떤 옵션을 선택하시겠습니까? (Which option?)
```

**분리된 HEAD (Detached HEAD) — 정확히 다음 2가지 옵션을 제시하세요:**

```
구현이 완료되었습니다. 현재 분리된 HEAD(외부에서 관리되는 작업 공간) 상태입니다. (Implementation complete. You're on a detached HEAD (externally managed workspace).)

1. 새 브랜치로 푸시 후 Pull Request 생성 (Push as new branch and create a Pull Request)
2. 현재 상태로 유지 (나중에 직접 처리) (Keep as-is (I'll handle it later))

어떤 옵션을 선택하시겠습니까? (Which option?)
```

메뉴는 위 목록에 있는 옵션만 사용하여 간결하고 정확하게 그대로 제시하세요. 작업을 폐기(discard)하는 것은 인간 파트너(human partner)가 명시적으로 요청한 경우에만 수행합니다(아래 "인간 파트너가 작업 폐기를 요청하는 경우" 참조). 파트너의 답변을 기다리세요. 통합(integration) 방식에 대한 결정권은 인간 파트너에게 있습니다.

## 5단계: 선택 실행 (Step 5: Execute Choice)

### 옵션 1: 로컬에서 병합 (Option 1: Merge Locally)

```bash
# 작업 디렉토리(CWD) 안전성을 위해 메인 저장소 루트 경로 획득
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"

# 먼저 병합 수행 — 무언가를 삭제하기 전에 성공 여부부터 검증
git checkout <base-branch>
git pull
git merge <feature-branch>

# 병합된 결과물에서 테스트 검증
<test command>
```

병합된 결과물에서 테스트가 실패하는 경우: 즉시 멈추고, 워크트리와 브랜치를 그대로 유지한 채 원인을 조사하세요 — 아직 아무것도 푸시되지 않았으므로 병합은 로컬에만 존재하며 복구 가능합니다.

병합된 결과물의 테스트가 모두 통과(green)하면: 워크트리를 정리하고(6단계), 브랜치를 삭제합니다:

```bash
git branch -d <feature-branch>
```

### 옵션 2: 푸시 및 PR 생성 (Option 2: Push and Create PR)

```bash
git push -u origin <feature-branch>
# 분리된 HEAD(detached HEAD) 상태라면 원격에 새 브랜치 이름을 지정합니다:
# git push origin HEAD:refs/heads/<new-branch>
```

그런 다음 호스팅 플랫폼(forge)의 도구 — 사용 가능한 CLI가 있다면 CLI, 또는 푸시할 때 대부분의 호스팅 플랫폼이 출력해 주는 생성 URL — 를 사용하여 `<base-branch>`를 대상으로 Pull/Merge Request를 생성하세요. 저장소에 PR 템플릿과 컨벤션이 있다면 이를 따르고, 생성된 URL을 인간 파트너에게 보고합니다.

워크트리는 유지하세요 — 인간 파트너가 PR 피드백을 반영하며 반복 작업할 때 그곳을 사용합니다.

### 옵션 3: 현재 상태로 유지 (Option 3: Keep As-Is)

보고: "`<name>` 브랜치를 유지합니다. 워크트리는 `<path>`에 보존되어 있습니다." ("Keeping branch `<name>`. Worktree preserved at `<path>`.")

### 인간 파트너가 작업 폐기를 요청하는 경우 (If your human partner asks to discard the work)

이 경로는 작업을 버리겠다는 명시적인 요청에 대한 응답으로만 존재합니다. 먼저 확인을 받으세요:

```
다음 항목들이 영구적으로 삭제됩니다 (This will permanently delete):
- 브랜치 (Branch) <name>
- 모든 커밋 (All commits): <commit-list>
- 워크트리 위치 (Worktree at) <path>

확인하려면 'discard'를 입력해 주세요. (Type 'discard' to confirm.)
```

정확히 그 확인 응답이 올 때까지 기다리세요. 응답이 도착하면:

```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"
```

그런 다음 워크트리를 정리하고(6단계), 브랜치를 강제 삭제합니다:

```bash
git branch -D <feature-branch>
```

## 6단계: 작업 공간 정리 (Step 6: Cleanup Workspace)

**옵션 1과 확인된 폐기(discard)의 경우에만 실행합니다.** 옵션 2와 옵션 3은 항상 워크트리를 보존합니다. 두 호출 경로 모두 이미 메인 저장소 루트로 디렉토리를 변경한 상태이며 — 워크트리 제거는 반드시 해당 워크트리 외부에서 실행해야 합니다 — 디렉토리 변경 전인 2단계에서 캡처해 둔 `GIT_DIR`/`GIT_COMMON`/`WORKTREE_PATH` 값을 사용합니다.

**`GIT_DIR == GIT_COMMON`인 경우:** 일반 저장소이므로 정리할 워크트리가 없습니다. 완료.

**`WORKTREE_PATH`가 `.worktrees/` 또는 `worktrees/` 아래에 있는 경우:** Superpowers가 생성한 워크트리이므로 우리가 정리를 담당합니다:

```bash
git worktree remove "$WORKTREE_PATH"
git worktree prune  # 자가 치유(Self-healing): 오래된(stale) 워크트리 등록 정보 정리
```

**제거가 거부된 경우** (`contains modified or untracked files`): 워크트리에 다른 곳에는 없는 파일들 — 커밋되지 않은 계획, 메모, 또는 임시 작업물 — 이 남아 있는 것입니다. 절대 임의로 `--force`를 사용하지 마세요. 인간 파트너에게 어떤 파일이 걸려 있는지 보여주고 질문하세요:

```bash
git -C "$WORKTREE_PATH" status --porcelain -uall
```

```
워크트리 제거가 거부되었습니다 — 다음 파일들이 아직 커밋되지 않았습니다 (Worktree removal refused — these files were never committed):

<file list>

1. 정리하기 전에 <branch>에 커밋 (Commit them to <branch> before cleanup)
2. <main repo root>로 이동 (Move them into <main repo root>)
3. 삭제 (복구 불가) (Delete them (unrecoverable))

어떻게 할까요? (Which?)
```

선택된 조치를 수행한 후 워크트리를 제거합니다.

**그 외의 경우 (Otherwise):** 호스트 환경이 이 작업 공간을 소유하고 있으므로 그대로 두세요. 플랫폼에서 작업 공간 종료(workspace-exit) 도구를 제공한다면 그것을 사용하세요.

## 빠른 참조 (Quick Reference)

| 옵션 (Option) | 병합 (Merge) | 푸시 (Push) | 워크트리 유지 (Keep Worktree) | 브랜치 정리 (Cleanup Branch) |
|--------|-------|------|---------------|----------------|
| 1. 로컬에서 병합 (Merge locally) | 예 (yes) | - | - | 예 (yes) |
| 2. PR 생성 (Create PR) | - | 예 (yes) | 예 (yes) | - |
| 3. 현재 상태로 유지 (Keep as-is) | - | - | 예 (yes) | - |
| 폐기 (명시적 요청 시에만) (Discard) | - | - | - | 예 (강제 삭제, force) |

## 흔한 합리화 (Common Rationalizations)

| 변명 (Excuse) | 현실 (Reality) |
|--------|---------|
| "이번 세션 초반에 이미 테스트가 통과했어" | 지금 통합하려는 바로 그 트리에서 테스트 스위트를 실행하세요. 통과한 실행 결과는 오직 그 테스트가 실행된 트리만 증명합니다. |
| "당연히 병합하길 원하겠지" | 통합 방식은 인간 파트너의 결정 사항입니다. 메뉴를 제시하고 기다리세요. |
| "이 기능은 이제 다 끝난 것 같으니 폐기(discard) 옵션도 제안해야지" | 메뉴는 적혀 있는 그대로가 완성본입니다. 폐기는 인간 파트너가 직접적인 표현으로 요청했을 때만 이루어집니다. |
| "'응, 지워버려(Yeah, get rid of it)'라고 했으니 확인받은 셈이지" | 오직 직접 입력된 `discard`라는 단어만이 삭제를 승인합니다. |
| "PR을 올렸으니 이제 워크트리는 거추장스러운 쓰레기야" | PR 피드백 수정은 그 워크트리에서 이루어집니다. 작업이 최종 반영될 때까지 워크트리는 유지됩니다. |
| "저기 다른 워크트리도 오래된 것 같은데 같이 정리해 줄까" | 오직 `.worktrees/` 또는 `worktrees/` 아래의 워크트리만 정리하세요. 그 외의 모든 것은 호스트의 소유입니다. |
| "제거가 거부됐네 — `--force`로 깔끔하게 마무리하자" | 제거 거부는 그 워크트리에만 존재하는 파일이 있다는 뜻입니다. `--force`는 그 파일들을 영구적으로 파괴합니다. 인간 파트너에게 보여주고 질문하세요. |
| "병합 결과물에서 난 테스트 실패는 아마 일시적인 오류(flaky)일 거야" | 병합 결과물에서 테스트가 실패하면 모든 것을 중단해야 합니다. 원인을 조사하는 동안 브랜치와 워크트리는 그대로 유지합니다. |
| "베이스 브랜치는 보나 마나 main이겠지" | 분기 지점(fork point)을 확인하거나 질문하세요. 잘못된 베이스에 병합하면 되돌리는 데 큰 비용이 듭니다. |
| "푸시가 거부됐네 — 강제 푸시(force-push)하면 해결될 거야" | 푸시가 거부되었다는 것은 원격 브랜치가 이동했다는 뜻입니다. 원인을 조사하세요. 강제 푸시는 오직 인간 파트너의 명시적인 요청이 있을 때만 수행합니다. |
