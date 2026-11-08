# Superpowers 테스트하기

Superpowers에는 서로 다른 두 종류의 테스트가 있으며, 각각 별도의 디렉토리에 있습니다:

- **`tests/`** — 플러그인의 비-LLM(non-LLM) 코드가 정상 작동하는가? 브레인스톰 서버 JS, OpenCode 플러그인 로딩, codex-plugin 동기화 및 분석 유틸리티에 대한 Bash + node + python 통합 테스트입니다.
- **`evals/`** — 실제 LLM 세션에서 에이전트가 올바르게 행동하는가? Claude Code / Codex / Gemini CLI의 실제 tmux 세션을 구동하고, LLM 액터(actor)와 검증기(verifier)가 스킬 준수 여부를 판정하는 Python 하네스입니다.

## 플러그인 테스트 (Plugin tests)

`tests/`에 위치합니다. 현재 포함된 항목은 다음과 같습니다:

- `tests/brainstorm-server/` — 브레인스톰 서버 JS 코드를 위한 node 테스트 스위트.
- `tests/opencode/` — OpenCode 플러그인 로딩, 부트스트랩 캐싱 및 도구 등록을 위한 bash 테스트.
- `tests/codex-plugin-sync/` — bash 동기화 검증.
- `tests/kimi/` — Kimi 플러그인 매니페스트 연결(wiring)을 위한 bash/Python 검사.
- `tests/claude-code/test-helpers.sh`, `analyze-token-usage.py` — 나머지 bash 테스트들에서 사용하는 유틸리티.
- `tests/claude-code/test-subagent-driven-development.sh` — 에이전트가 SDD(서브에이전트 기반 개발)를 설명할 수 있는지 확인하는 테스트(quorum 대응 항목 없음; 행동이 아닌 설명 기억 여부를 테스트함).
- `tests/claude-code/test-subagent-driven-development-integration.sh` — 토큰 분석이 포함된 확장 SDD 통합 테스트(quorum은 YAGNI 하위 집합을 다루며, bash는 커밋 수, Claude Code 작업 추적 및 토큰 텔레메트리 검증을 추가함).
- `tests/claude-code/test-worktree-native-preference.sh` — worktree 스킬에 대한 RED-GREEN-REFACTOR 검증(quorum은 PRESSURE 단계를 다루며, bash는 RED/GREEN 베이스라인도 함께 다룸).
- `tests/explicit-skill-requests/` — quorum에서 다루지 않는 Haiku 전용, 멀티턴(multi-turn), 스킬 이름 직접 지명 프롬프트 테스트.
- `tests/diagnosing-superpowers/test-skill-structure.sh` — diagnosing-superpowers 스킬에 대한 구조적 검사(프론트매터, 참조 파일, 유출 스캔, 단어 수 예산); 행동 시나리오 평가(eval) 기록은 메인테이너가 저장소 외부에서 관리합니다.

플러그인 테스트는 해당 디렉토리의 `run-*.sh` 또는 `npm test`를 통해 실행하세요.

## 스킬 행동 평가 (Skill behavior evals)

`evals/`(기존 Drill에서 이름이 변경된 [superpowers-evals](https://github.com/prime-radiant-inc/superpowers-evals/) 평가 랩)에 위치합니다. Quorum은 시스템의 일부인 하네스 CLI로, Gauntlet QA 에이전트를 통해 실제 코딩 에이전트 CLI들을 구동하고 각 시나리오의 인수 기준(acceptance criteria)과 결정론적 사후 검사(deterministic post-checks)에 따라 채점합니다. 시나리오는 `evals/scenarios/<name>/`에 있습니다. 설정 방법, 컨테이너 런타임 및 안전 모델에 대해서는 `evals/README.md`를 참조하세요. 빠른 시작(로컬 비상 실행):

```bash
cd evals
bun install
export SUPERPOWERS_ROOT=/path/to/superpowers
bun run quorum run scenarios/triggering-test-driven-development --coding-agent claude
bun run quorum show <run-dir>
```

Quorum 시나리오는 실행 속도가 느리며(각 3~30분 이상 소요) 허용적인(permissive) 모드에서 실제 LLM 세션을 실행합니다 — 먼저 `evals/README.md`의 Live Eval Risk 섹션을 읽으세요. 공개 CI에서는 정적 게이트(`bun run check`, `bun run quorum check`)만 안전하게 실행할 수 있으며, 자연스러운 후속 운영 방식은 계층형 모델(PR 시 정적 게이트 실행, 야간 및 온디맨드로 라이브 전체 실행)입니다.
