# Claude Code 스킬 테스트 (Skills Tests)

Claude Code CLI를 사용하는 superpowers 스킬 자동화 테스트입니다.

## 개요 (Overview)

이 테스트 스위트는 스킬이 올바르게 로드되고 Claude가 예상대로 스킬을 따르는지 검증합니다. 테스트는 헤드리스(headless) 모드(`claude -p`)로 Claude Code를 호출하여 동작을 확인합니다.

## 요구사항 (Requirements)

- Claude Code CLI가 설치되어 있고 PATH에 등록되어 있어야 함 (`claude --version`이 작동해야 함)
- 로컬 superpowers 플러그인이 설치되어 있어야 함 (설치 방법은 메인 README 참조)

## 테스트 실행하기 (Running Tests)

### 모든 빠른 테스트 실행 (권장):
```bash
./run-skill-tests.sh
```

### 통합 테스트 실행 (느림, 10~30분 소요):
```bash
./run-skill-tests.sh --integration
```

### 특정 테스트 실행:
```bash
./run-skill-tests.sh --test test-subagent-driven-development.sh
```

### 상세 출력(verbose)과 함께 실행:
```bash
./run-skill-tests.sh --verbose
```

### 사용자 정의 타임아웃 설정:
```bash
./run-skill-tests.sh --timeout 1800  # 통합 테스트용 30분
```

## 테스트 구조 (Test Structure)

### test-helpers.sh
스킬 테스트를 위한 공통 함수:
- `run_claude "prompt" [timeout]` - 프롬프트로 Claude 실행
- `assert_contains output pattern name` - 패턴이 존재하는지 검증
- `assert_not_contains output pattern name` - 패턴이 존재하지 않는지 검증
- `assert_count output pattern count name` - 정확한 출현 횟수 검증
- `assert_order output pattern_a pattern_b name` - 출현 순서 검증
- `create_test_project` - 임시 테스트 디렉토리 생성
- `create_test_plan project_dir` - 샘플 계획 파일 생성

### 테스트 파일 (Test Files)

각 테스트 파일은 다음을 수행합니다:
1. `test-helpers.sh`를 로드(source)합니다
2. 특정 프롬프트로 Claude Code를 실행합니다
3. 어설션(assertion)을 사용하여 예상 동작을 검증합니다
4. 성공 시 0, 실패 시 0이 아닌 값을 반환합니다

## 예제 테스트 (Example Test)

```bash
#!/usr/bin/env bash
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
source "$SCRIPT_DIR/test-helpers.sh"

echo "=== Test: My Skill ==="

# Claude에게 스킬에 대해 질문
output=$(run_claude "What does the my-skill skill do?" 30)

# 응답 검증
assert_contains "$output" "expected behavior" "Skill describes behavior"

echo "=== All tests passed ==="
```

## 현재 포함된 테스트 (Current Tests)

### 빠른 테스트 (기본으로 실행됨)

#### test-subagent-driven-development.sh
스킬 내용과 요구사항을 테스트합니다(약 2분 소요):
- 스킬 로딩 및 접근성
- 워크플로우 순서 (코드 품질 검토 전에 스펙 준수 검토 수행)
- 자체 리뷰(self-review) 요구사항 문서화 여부
- 계획 읽기 효율성 문서화 여부
- 스펙 준수 리뷰어의 회의적 태도(skepticism) 문서화 여부
- 리뷰 루프 문서화 여부
- 작업 컨텍스트 제공 문서화 여부

### 통합 테스트 (--integration 플래그 사용)

#### test-subagent-driven-development-integration.sh
전체 워크플로우 실행 테스트입니다(약 10~30분 소요):
- Node.js 설정이 포함된 실제 테스트 프로젝트 생성
- 2개의 작업으로 구성된 구현 계획 생성
- subagent-driven-development를 사용하여 계획 실행
- 실제 동작 검증:
  - 시작 시 계획을 한 번만 읽음 (작업마다 읽지 않음)
  - 서브에이전트 프롬프트에 전체 작업 텍스트 제공
  - 서브에이전트가 보고 전에 자체 리뷰 수행
  - 코드 품질 검토 전에 스펙 준수 리뷰 수행
  - 스펙 리뷰어가 독립적으로 코드를 읽음
  - 작동하는 구현물이 생성됨
  - 테스트 통과
  - 적절한 git 커밋 생성

**테스트 대상:**
- 워크플로우가 실제로 엔드투엔드(end-to-end)로 작동하는지 여부
- 개선 사항이 실제로 적용되었는지 여부
- 서브에이전트가 스킬을 올바르게 따르는지 여부
- 최종 코드가 정상 작동하고 테스트되었는지 여부

#### test-worktree-native-preference.sh
using-git-worktrees 스킬에 대한 RED-GREEN-REFACTOR 검증입니다(약 5분 소요):
- RED: Step 1a가 없는 스킬 — 에이전트가 `git worktree add`를 사용해야 함
- GREEN: Step 1a가 있는 스킬 — 에이전트가 네이티브 EnterWorktree 도구를 사용해야 함
- PRESSURE: 기존 `.worktrees/`가 있고 시간적 압박이 주어진 상황에서 GREEN과 동일하게 동작해야 함
- Drill 시나리오 `worktree-creation-under-pressure.yaml`은 PRESSURE 단계만 다룸

## 새 테스트 추가하기 (Adding New Tests)

1. 새 테스트 파일 생성: `test-<skill-name>.sh`
2. `test-helpers.sh` 로드(source)
3. `run_claude`와 어설션을 사용하여 테스트 작성
4. `run-skill-tests.sh`의 테스트 목록에 추가
5. 실행 권한 부여: `chmod +x test-<skill-name>.sh`

## 타임아웃 고려사항 (Timeout Considerations)

- 기본 타임아웃: 테스트당 5분
- Claude Code가 응답하는 데 시간이 걸릴 수 있음
- 필요한 경우 `--timeout`으로 조정
- 실행 시간이 너무 길어지지 않도록 테스트의 초점을 명확히 할 것

## 실패한 테스트 디버깅하기 (Debugging Failed Tests)

`--verbose`를 사용하면 전체 Claude 출력을 확인할 수 있습니다:
```bash
./run-skill-tests.sh --verbose --test test-subagent-driven-development.sh
```

verbose 옵션이 없으면 실패한 경우에만 출력이 표시됩니다.

## CI/CD 통합 (CI/CD Integration)

CI에서 실행하려면:
```bash
# CI 환경을 위해 명시적인 타임아웃과 함께 실행
./run-skill-tests.sh --timeout 900

# 종료 코드 0 = 성공, 0이 아닌 값 = 실패
```

## 참고사항 (Notes)

- 기본 테스트는 전체 실행이 아니라 스킬 *지침(instructions)*을 검증합니다
- 전체 워크플로우 테스트는 매우 느릴 수 있습니다
- 핵심 스킬 요구사항을 검증하는 데 집중하세요
- 테스트는 결정론적(deterministic)이어야 합니다
- 구현 세부사항을 테스트하는 것은 피하세요
