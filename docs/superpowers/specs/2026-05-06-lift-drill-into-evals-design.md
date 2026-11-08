# drill을 superpowers의 `evals/`로 통합 — 설계 (Lift drill into superpowers as `evals/` — design)

## 배경

Drill은 `obra/drill`이라는 자체 저장소에 있는 Python 기반 스킬 준수(skill-compliance) 벤치마크 도구입니다. 실제 tmux 세션을 구동하고, 시뮬레이션된 사용자로서 LLM 액터(actor)를 실행하며, 결과 트랜스크립트에 대해 LLM 검증기(verifier)를 실행하여 시나리오별 통과/실패 여부를 보고합니다. Claude Code, Codex, Gemini CLI 및 (최근 커밋 기준) OpenCode와 Copilot CLI를 지원합니다.

Drill은 이미 superpowers의 *사실상의(de facto)* 평가(eval) 하네스입니다. drill 저장소의 PRI-1397 커밋 시리즈는 약 22개의 superpowers bash 테스트를 drill 시나리오로 이식했으며, 가장 최근의 superpowers 커밋(`a2292c5`)은 *"replaced by drill behavioral coverage"*라는 메시지와 함께 중복된 bash 테스트를 명시적으로 제거했습니다. 마이그레이션 흐름은 이미 시작되었으며, 이 스펙은 이를 완성합니다.

이 작업은 drill을 superpowers의 `evals/` 아래로 이동시키고, drill 시나리오 커버리지에 대한 파일별 검증을 거친 후 중복된 bash 테스트를 삭제하며, 기여자들이 새로운 구조에 잘 적응할 수 있도록 문서를 업데이트합니다.

## 목표

1. `evals/`를 superpowers의 공식(canonical) 평가 하네스로 만듭니다 — 전체 drill 소스 코드, 시나리오, 픽스처, 프롬프트, 백엔드 설정 및 테스트 포함.
2. `superpowers/tests/`에 있는 bash 테스트 중 drill 시나리오에 의해 100% 커버되는 것으로 개별 검증된 테스트는 삭제하고, 나머지는 보존합니다.
3. `tests/`(플러그인 인프라: bash + node + python 통합 테스트)와 `evals/`(액터 + 검증기를 활용한 LLM 행동 평가) 사이의 역할 분담을 명확히 하고 문서화합니다.
4. 최상위 문서들(`README.md`, `CLAUDE.md`, `docs/testing.md`)이 기여자들에게 올바른 위치를 안내하도록 합니다.
5. 독립된 `obra/drill` 저장소는 계속 존재하며(이 PR에서는 건드리지 않음), 이 PR이 병합된 후 별도의 수동 단계를 통해 아카이브(archive) 처리됩니다.

## 비목표 (Non-goals)

- **CI 통합.** 여기서는 수동 실행만 다룹니다. 자연스러운 후속 작업은 "계층화(tiered)"입니다: 모든 PR마다 빠른 하위 집합을 실행하고, 매일 밤 및 온디맨드로 전체 테스트를 실행하는 것입니다. 이를 위해서는 API 예산 결정, GitHub Actions 시크릿, 그리고 `tmux` + `node` + `python` + `claude` / `codex` / `gemini` CLI가 설치된 러너 이미지가 필요합니다. 이번 범위에서는 제외됩니다.
- **스킬과 시나리오의 동일 위치 배치(co-location).** 시나리오들은 `evals/scenarios/`에 중앙 집중식으로 유지됩니다. 만약 나중에 각 스킬이 자체 시나리오를 소유해야 한다고 결정하더라도, 이는 경로 찾기 및 이름 바꾸기 작업일 뿐이며 YAML 포맷은 변하지 않습니다.
- **내부 Python 패키지 이름 변경** (`drill` → `evals`). 사용자에게 보이는 디렉터리 이름은 `evals/`이지만, diff 크기를 작게 유지하기 위해 Python 패키지는 `drill`이라는 이름을 그대로 유지합니다. `evals/README.md`에 짧은 설명을 추가합니다.
- **Drill 저장소 아카이브.** 이 PR은 `obra/drill`을 건드리지 않습니다. 병합 후 drill 저장소는 수동으로 아카이브 처리됩니다 (GitHub에서 읽기 전용으로 전환, README에 `obra/superpowers/evals/`로 향하는 포인터 추가).
- **`tests/claude-code/analyze-token-usage.py`를 `evals/bin/`으로 이동.** 유용한 유틸리티이지만 테스트 코드는 아닙니다. 나중에 이동할 수 있으며 이 PR에서 필수는 아닙니다.

## 브랜치 전략 (Branching)

`dev`에서 `f/evals-lift`로 브랜치를 분기합니다. 이 작업은 현재 열려 있는 `f/cross-platform` PR과 독립적입니다 — `README.md` 외에는 공유하는 파일 변경 사항이 없으며, 충돌이 발생하더라도 병합 시점에 해결하기에 충분히 작습니다.

## 이동 후 아키텍처

```
superpowers/
  evals/                              ← 신규 (전체 drill 복사본)
    pyproject.toml                    (Python 3.11, uv 관리형)
    uv.lock
    .gitignore                        (drill 자체 설정; results/, .venv/, .env)
    README.md                         (기존 drill의 README; 설치 지침 업데이트됨)
    CLAUDE.md                         (기존 drill의 CLAUDE.md; 경로 업데이트됨)
    docs/
      design.md                       (drill의 설계 문서 — 원문 그대로 보존, 이 스펙에서 상호 링크됨)
      manual-testing.md
      pressure-and-red-testing.md
    drill/                            (Python 패키지; 이름 유지; cli, engine, actor, verifier 등)
    backends/                         (claude-*.yaml, codex.yaml, gemini.yaml)
    scenarios/                        (32개 이상의 YAML 시나리오)
    setup_helpers/                    (15개의 Python 헬퍼; create_base_repo, sdd_*, spec_*, worktree 등)
    fixtures/                         (template-repo, sdd-go-fractals, sdd-svelte-todo)
    prompts/                          (actor.md, verifier.md)
    bin/                              (단언(assertion) 헬퍼 스크립트: tool-called, tool-count 등)
    tests/                            (drill 자체 pytest 스위트)

  tests/                              ← bash 테스트는 기본적으로 보존됨
    brainstorm-server/                ← 유지 (brainstorm-server JS 코드용 node 테스트)
    opencode/                         ← 유지 (플러그인 로딩 테스트)
    codex-plugin-sync/                ← 유지 (동기화 검증)
    claude-code/                      ← 대부분 유지 — 삭제 게이트 참조
    explicit-skill-requests/          ← 대체가 검증되지 않는 한 유지
    skill-triggering/                 ← 대체가 검증되지 않는 한 유지
    subagent-driven-dev/              ← 대체가 검증되지 않는 한 유지

  docs/
    testing.md                        ← 업데이트됨 ("Plugin tests" + "Skill behavior evals"로 분리)
    superpowers/
      specs/
        2026-05-06-lift-drill-into-evals-design.md   ← 본 스펙

  README.md                           ← Contributing 섹션에 evals/를 가리키는 작은 포인터 추가
  CLAUDE.md                           ← "Eval harness lives at evals/" 한 줄 포인터 추가
```

이 PR 이후 `tests/`와 `evals/` 디렉터리는 명확하게 구분되는 역할을 수행합니다:

- **`tests/`** — 플러그인의 비-LLM 코드가 작동하는가? brainstorm-server JS 코드, OpenCode 플러그인 로딩, codex-plugin-sync 동기화 검증을 위한 단위 및 통합 테스트입니다. Bash + node + python으로 구성됩니다.
- **`evals/`** — 실제 LLM 세션에서 에이전트가 올바르게 행동하는가? 액터와 검증기가 포함된 Drill 시나리오입니다. Python 전용이며 실제 tmux 세션을 실행합니다.

## 삭제 게이트 (각 bash 테스트별)

bash 테스트는 drill 시나리오가 해당 테스트의 모든 단언(assertion)을 커버한다는 것이 *검증된 경우에만* 삭제됩니다. 구현 계획은 파일별로 이 검증 내역을 문서화합니다: bash 테스트를 읽고, 검사 항목들을 나열하고, 대응하는 drill 시나리오를 찾아 각 검사 항목이 일치하는 `verify.assertions` 또는 `verify.criteria` 항목을 가지고 있는지 확인합니다. 단 하나의 검사 항목이라도 누락되어 있다면, drill 시나리오를 확장하거나 bash 테스트를 유지하는 것 중 하나를 선택해야 합니다. 기본값은 유지(keep)입니다.

**잠정적 커버리지 맵** (커밋 메시지 기반이며, 실제 삭제 전에는 파일별 검증이 필요함):

| Bash 테스트 | 주장된 drill 대체물 | 커버리지 상태 |
|---|---|---|
| `tests/skill-triggering/prompts/*` (프롬프트 파일 6개) | `triggering-*.yaml` (시나리오 6개) | 후보 — 삭제 전 프롬프트별 검증 필요 |
| `tests/skill-triggering/run-test.sh`, `run-all.sh` | 해당 없음 (테스트가 아닌 러너) | **유지** — 러너 스크립트 |
| `tests/explicit-skill-requests/prompts/please-use-brainstorming.txt` | 검증 필요 — 아직 drill에 명확한 대응물이 없음 | drill 시나리오가 추가되지 않는 한 **유지** 가능성 높음 |
| `tests/explicit-skill-requests/prompts/use-systematic-debugging.txt` | 검증 필요 — drill에 명확한 대응물이 없음 | drill 시나리오가 추가되지 않는 한 **유지** 가능성 높음 |
| `tests/explicit-skill-requests/run-claude-describes-sdd.sh` | 부분적 → `mid-conversation-skill-invocation.yaml` | 후보 — 스크립트별 검증 필요 |
| `tests/explicit-skill-requests/run-haiku-test.sh` | Haiku 전용 동작을 커버하는 drill 시나리오 없음 | **유지** |
| `tests/explicit-skill-requests/run-multiturn-test.sh`, `run-extended-multiturn-test.sh` | 다중 턴 빌드업을 커버하는 drill 시나리오 없음 | drill 시나리오가 추가되지 않는 한 **유지** |
| `tests/explicit-skill-requests/run-test.sh`, `run-all.sh` | 해당 없음 (러너) | **유지** |
| `tests/subagent-driven-dev/go-fractals/`, `tests/subagent-driven-dev/svelte-todo/` | `sdd-go-fractals.yaml`, `sdd-svelte-todo.yaml` | 후보 — 삭제 전 검증 필요 (테스트 스위트 통과에 대한 실제 단언들이 포함되어 있음) |
| `tests/claude-code/test-document-review-system.sh` | `spec-reviewer-catches-planted-flaws.yaml` | 후보 — 삭제 전 검증 필요 |
| `tests/claude-code/test-requesting-code-review.sh` | `code-review-catches-planted-bugs.yaml` | 후보 — 삭제 전 검증 필요 |
| `tests/claude-code/test-subagent-driven-development-integration.sh` | `sdd-rejects-extra-features.yaml` (YAGNI 하위 집합) | **부분적** — bash 테스트는 최소 3개 이상의 커밋 / `npm test` 통과 / `analyze-token-usage.py` 실행도 함께 검증함. Drill 시나리오는 금지된 export 및 게이트로서의 리뷰어를 검증함. 대부분 겹치지 않으므로 거의 확실하게 **유지 + drill 시나리오 확장**해야 함. |
| `tests/claude-code/test-subagent-driven-development.sh` | 메타/문서화 테스트 (에이전트에게 SDD를 *설명*하라고 요청함); 설명 테스트를 커버하는 drill 시나리오 없음 | drill 시나리오가 추가되지 않는 한 **유지** |
| `tests/claude-code/test-worktree-native-preference.sh` | `worktree-creation-under-pressure.yaml` | 후보 — 삭제 전 검증 필요 |
| `tests/claude-code/test-helpers.sh`, `run-skill-tests.sh`, `analyze-token-usage.py` | 해당 없음 (테스트가 아닌 유틸리티) | **유지** — 라이브러리/도구 |

## 검증 프로토콜 (서브에이전트 게이트 방식)

구현 계획의 모든 변경 사항은 커밋 전에 독립적인 서브에이전트에 의해 교차 검증됩니다.

| 변경 카테고리 | 서브에이전트 검증 내용 |
|---|---|
| 각 bash 테스트 삭제 | 다음 입력과 함께 서브에이전트를 디스패치합니다: (a) bash 테스트 파일 내용, (b) 후보 drill 시나리오 YAML, (c) 프롬프트: *"bash 테스트가 수행하는 모든 단언(assertion)을 나열하세요. drill 시나리오의 모든 verify 항목을 나열하세요. 각 bash 단언에 대해 일치하는 drill 검사를 찾거나 일치 항목 없음으로 보고하세요. 단언별 비교 표를 출력하세요."* 서브에이전트의 출력이 곧 게이트가 됩니다 — 모든 bash 단언에 일치 항목이 있을 때만 삭제합니다. |
| 초기 `evals/` 복사 | 서브에이전트가 다음을 검증합니다: (a) 출처를 감사(audit)할 수 있도록 복사 시점의 drill SHA가 커밋 메시지에 기록되었는지; (b) (단순 파일 개수가 아니라) 모든 파일에 대해 **파일별 SHA-256 체크섬**이 drill 저장소와 일치하는지; (c) 제외 경로(`.git/`, `.venv/`, `results/`, `.env`, `__pycache__/`, `*.egg-info/`, 모든 `.private-journal/`)가 `evals/`에 없는지; (d) 모든 백엔드 YAML이 이동 후에도 존재하는 경로를 참조하는지; (e) `pyproject.toml`, `uv.lock`, `.gitignore`가 온전한지. |
| Drill 자체 pytest 스위트 | 경로 기본값 변경 후 서브에이전트가 `cd evals && uv run pytest`를 실행합니다. Drill은 `SUPERPOWERS_ROOT` 환경 변수 동작을 테스트하는 `test_backend.py`를 포함하여 `evals/tests/`에 자체 pytest 스위트를 포함하고 있습니다 — 이 테스트들은 새로운 헬퍼에 맞게 업데이트되어야 하며 계속 통과해야 합니다. |
| 삭제 후 참조 정리 (Reference scrubbing) | 서브에이전트가 (`node_modules/`, `.venv/`, `evals/`를 제외한) 전체 superpowers 트리에서 삭제된 bash 테스트 경로에 대한 참조를 grep으로 검색합니다. 검색 대상: `docs/`, `docs/superpowers/plans/`, `RELEASE-NOTES.md`, `CLAUDE.md`, `GEMINI.md`, `AGENTS.md`, `README.md`, `.github/`, `scripts/`, `.opencode/INSTALL.md`, `.codex-plugin/INSTALL.md`, `lefthook.yml`. 발견된 모든 항목은 업데이트되거나 누락된 의존성을 드러내게 됩니다. |
| 경로 기본값 변경 (`SUPERPOWERS_ROOT` 기본값) | 경로 변경 후 서브에이전트가 최소 하나 이상의 비용이 적게 드는 drill 시나리오(예: `triggering-test-driven-development`)를 실행하고 여전히 통과하는지 확인합니다. 단순 코드 리뷰가 아닌 실제 검증입니다. |
| PR 전 최종 적대적 리뷰 (Adversarial review) | 두 개의 서브에이전트를 병렬로 실행하며, 크로스 플랫폼 PR에서 사용했던 것과 동일한 "가장 타당한 이슈를 많이 찾는 쪽에 5점 부여" 프레이밍을 적용합니다. 소스 코드와 동작을 모두 검증합니다. |

각 서브에이전트 작업은 명시적인 입력과 통과 기준과 함께 구현 계획에 개별 항목으로 등록됩니다. 추적 기록을 감사할 수 있도록 서브에이전트의 출력은 관련 커밋 메시지에 요약됩니다("Subagent verification: …").

## 구체적인 경로/설정 수정 사항

**이 스펙을 작성하기 전에 사전 검증되었습니다.** `drill/cli.py`는 `PROJECT_ROOT = Path(__file__).parent.parent`를 정의합니다. 이동 후 `cli.py`는 `evals/drill/cli.py`에 위치하게 되므로, `PROJECT_ROOT`는 `evals/`로 해석되고 `PROJECT_ROOT.parent`는 superpowers 저장소 루트로 해석됩니다. 이것이 바로 `SUPERPOWERS_ROOT`가 기본값으로 가져야 할 값입니다.

**YAML 치환 감사.** 네 개의 `claude*.yaml` 백엔드 설정만이 (`--plugin-dir` 플래그를 위해) `args`에 `${SUPERPOWERS_ROOT}`를 보간(interpolate)합니다. `codex.yaml`과 `gemini.yaml`은 `required_env`에만 `SUPERPOWERS_ROOT`를 나열합니다 (실행 전/후 훅에서 `engine.py:233` / `setup.py:25`의 `os.environ["SUPERPOWERS_ROOT"]` 조회에 의해 소비됨). 헬퍼의 `os.environ` 변경은 이 두 코드 경로를 모두 커버합니다.

| 파일 | 현재 상태 | 변경 후 |
|---|---|---|
| `drill/cli.py` | 모듈 임포트 시 `load_dotenv(PROJECT_ROOT / ".env")` 실행; `SUPERPOWERS_ROOT`에 대한 처리 없음 | `load_dotenv` 이후에 아직 설정되지 않은 경우에만 `os.environ["SUPERPOWERS_ROOT"]`를 `str(PROJECT_ROOT.parent)`로 설정하는 새 헬퍼 `_set_superpowers_root_default()`를 호출합니다. 순서: `load_dotenv` → 기본값 설정 → click 그룹 정의. |
| `drill/engine.py:233`, `drill/setup.py:25` | 직접적인 `os.environ["SUPERPOWERS_ROOT"]` 접근 (미설정 시 KeyError) | 변경 없음. CLI 시작 훅이 엔진/설정이 실행될 시점에는 환경 변수가 설정되어 있음을 보장합니다. |
| `backends/claude*.yaml` (5개 파일) | `--plugin-dir`를 위해 `args`에서 `${SUPERPOWERS_ROOT}` 치환 | 변경 없음. YAML 치환은 CLI 시작 이후인 백엔드 로드 시점에 `os.environ`을 읽습니다. |
| `backends/codex.yaml`, `backends/gemini.yaml` | `required_env`에만 `SUPERPOWERS_ROOT` 포함 | `required_env`에서 제거합니다 (헬퍼가 값을 공급하므로). `claude*.yaml`은 하위 호환성을 위해 `required_env`를 유지합니다 (환경 변수가 오버라이드로 작동함). |
| `evals/tests/test_backend.py` | `SUPERPOWERS_ROOT`가 `required_env` 목록에 있는지 검증하는 테스트 및 경로 해석 테스트 | 새로운 계약에 맞게 테스트를 업데이트합니다: 헬퍼가 공급하는 기본값, 환경 변수 오버라이드 정상 작동 여부, codex/gemini에서 `required_env`가 더 이상 필수가 아닌 점 반영. |
| `evals/README.md` | "export SUPERPOWERS_ROOT=/path/to/superpowers" | export 줄을 제거합니다. 환경 변수가 자동으로 `evals/`의 부모 디렉터리를 기본값으로 삼는다는 점을 명시하고, 유일하게 필요한 설정은 `ANTHROPIC_API_KEY`(또는 `OPENAI_API_KEY` / Gemini 인증)임을 안내합니다. |
| `evals/CLAUDE.md` | 동일 | 동일 |
| `evals/.gitignore` | drill의 기존 패턴들 (`results/`, `.venv/`, `__pycache__/`, `.env`, `*.pyc`, `*.egg-info/`, `dist/`, `build/`, `.claude/`) | 원문 그대로 복사합니다. 패턴들은 파일 위치에 상대적이므로 `evals/` 아래에서 올바르게 적용됩니다. |
| `evals/lefthook.yml` | drill은 `pre-commit: uv run ruff check && uv run ty check`를 정의하는 `lefthook.yml`을 포함함 | `evals/lefthook.yml`로 이동합니다. (a) superpowers 루트에 lefthook을 설치하고 `evals/lefthook.yml`로 위임(federate)하게 하거나, (b) 기여자들이 수동으로 `cd evals && lefthook run pre-commit`을 실행하도록 문서화합니다. **구현 시 결정 사항: 단순성을 위해 옵션 (b) 선택** — superpowers의 최상위 워크플로우를 변경하지 않습니다. |

`.env` 위치: `evals/.env`(gitignored)를 유지합니다. 기여자들은 거기서 파일을 불러오거나 셸 환경에 `ANTHROPIC_API_KEY`를 설정합니다.

**작은 추가가 필요한 최상위 superpowers 파일들:**

- `superpowers/.gitignore`: `evals/results/`, `evals/.venv/`, `evals/.env`를 추가합니다 (이중 안전장치; 로컬에서는 `evals/.gitignore`가 이미 이를 커버함).
- `superpowers/CLAUDE.md`: 에이전트가 발견할 수 있도록 "Eval harness lives at `evals/` — see `evals/README.md`"라는 한 줄 포인터를 추가합니다.
- `superpowers/docs/testing.md`: "## Plugin tests"(삭제된 테스트 참조를 다듬은 기존 `tests/` 내용)와 "## Skill behavior evals"(한 단락 요약 + `evals/` 포인터)로 분리합니다.
- `superpowers/README.md`: Contributing 섹션에 스킬 행동 테스트를 위해 `evals/`를 안내하는 한 줄을 추가합니다.

## 마이그레이션 순서

각 단계는 별도의 커밋(또는 작은 커밋 그룹)으로 구성됩니다. 2단계가 가장 큰 단일 커밋(drill 원문 그대로 복사)이며, 이후 단계들은 작고 원자적입니다.

```
1. `dev`에서 브랜치 분기 (f/evals-lift)

2. drill 저장소를 evals/로 복사 (되돌리기 쉬운 단일 커밋)
   ├─ 복사 시점의 drill SHA 기록 → 커밋 메시지에 포함
   ├─ `rsync -a --exclude=.git --exclude=.venv --exclude=results
   │  --exclude=.env --exclude=__pycache__ --exclude='*.egg-info'
   │  --exclude=.private-journal /path/to/drill/ evals/` 사용
   │  (명시적 제외를 위해 `cp -r` 대신 rsync 선택;
   │  `find evals -name '.git' -type d`가 아무것도 반환하지 않는지 확인하여 검증)
   ├─ 서브에이전트 게이트: 제외되지 않은 모든 파일에 대해 파일별 SHA-256 체크섬이
   │  drill 저장소와 일치하는지 확인; 제외된 경로들이 evals/에 없는지 확인
   └─ 스모크 체크: `cd evals && uv sync` 성공 확인 (설치만 증명하며,
      행동 테스트는 아님)

3. 경로 기본값 업데이트
   ├─ drill/cli.py에 _set_superpowers_root_default() 헬퍼 추가
   ├─ load_dotenv 이후, click 그룹 정의 이전에 연결
   ├─ evals/README.md 및 evals/CLAUDE.md 업데이트 (SUPERPOWERS_ROOT 설치 단계 제거)
   ├─ codex.yaml/gemini.yaml의 required_env에서 SUPERPOWERS_ROOT 제거
   │  (claude*.yaml에서는 오버라이드용으로 유지)
   └─ 새로운 계약에 맞게 evals/tests/test_backend.py 업데이트

4. 새 위치에서 검증 (두 가지 확인)
   ├─ drill 자체 pytest 실행: `cd evals && uv run pytest` — 반드시 통과해야 함
   └─ 비용이 적게 드는 drill 시나리오 실행: `cd evals && uv run drill run
      triggering-test-driven-development -b claude` — 반드시 통과해야 함.
      단순 코드 리뷰가 아닌 실제 행동 검증.

5. Bash 테스트 삭제 단계 — 서브에이전트 게이트를 통한 파일별 진행
   후보 삭제 목록의 각 파일에 대해:
   a. 서브에이전트가 bash 테스트 단언(assertions)과 drill 시나리오 verify 블록을 비교
   b. 통과 기준: 모든 bash 단언에 일치하는 drill 검사가 존재함
   c. 통과 시 → bash 테스트 파일 삭제 (파일당 또는 응집력 있는 그룹당 하나의 커밋)
   d. 실패 시 → drill 시나리오를 확장하거나(별도 커밋 + 검증) bash 테스트를 유지함(커밋 없음)

6. 오래된 참조 정리 (Stale-reference scrub)
   ├─ 서브에이전트가 (node_modules/, .venv/, evals/를 제외한) superpowers 트리에서
   │  삭제된 파일 경로들을 grep으로 검색
   ├─ 검색 대상: docs/, docs/superpowers/plans/, RELEASE-NOTES.md,
   │  CLAUDE.md, GEMINI.md, AGENTS.md, README.md, .github/, scripts/,
   │  .opencode/INSTALL.md, .codex-plugin/INSTALL.md, lefthook.yml
   ├─ 활성 참조 업데이트 (예: docs/testing.md, README.md 설치 지침)
   └─ docs/superpowers/plans/*.md 및 RELEASE-NOTES.md의 과거 참조는
      다시 쓰는 대신 짧은 주석("(test removed; behavior covered by drill scenario X)")과
      함께 보존함 — 이들은 살아있는 문서가 아니라 날짜가 기록된 아티팩트이기 때문임.

7. 최상위 문서 업데이트
   ├─ docs/testing.md 분리
   ├─ CLAUDE.md 포인터 추가
   └─ README.md Contributing 섹션 업데이트

8. 스모크 체크 재실행 (회귀 게이트)
   ├─ `cd evals && uv run pytest`
   └─ `cd evals && uv run drill run triggering-test-driven-development -b claude`

9. 최종 적대적 리뷰 (Final adversarial review)
   └─ 두 개의 병렬 서브에이전트, 전체 diff 대상, "가장 타당한 이슈를 많이 찾는 쪽에
      5점 부여" 프레이밍. 푸시 전에 발견된 사항들을 해결함.

10. 브랜치 푸시 + dev 브랜치 대상으로 PR 열기
    └─ PR 설명에 포함할 내용: 복사 시점에 고정된 drill SHA, 아카이브 작업 항목
       ("병합 후: obra/drill 아카이브 처리, obra/superpowers/evals/를 가리키는 README 포인터 추가"),
       삭제된 파일별 커버리지 증명 내역.
```

## 검증 (구현 후)

구현 계획은 다음을 보여주어야 합니다:

- 2단계 이후 제외되지 않은 모든 drill 소스 파일이 `evals/`에 존재함 (`obra/drill@<recorded-sha>` 대비 서브에이전트 **파일별 SHA-256 체크섬 비교**).
- 제외 경로(`.git/`, `.venv/`, `results/`, `.env`, `__pycache__/`, `*.egg-info/`, `.private-journal/`)가 `evals/`에 없음.
- 2단계 커밋 메시지에 drill 소스 SHA가 기록됨.
- `SUPERPOWERS_ROOT`를 설정하지 않은 상태에서도 `cd evals && uv sync`가 성공함.
- `cd evals && uv run pytest`가 통과함 (drill 자체 pytest 스위트).
- `cd evals && uv run drill list`가 기록된 SHA의 독립 실행형 drill 저장소와 동일한 시나리오 개수를 반환함.
- `cd evals && uv run drill run triggering-test-driven-development -b claude`가 통과함 (경로 기본값이 엔드투엔드로 작동함을 증명).
- 삭제된 각 bash 테스트에 대해: 모든 단언이 drill 검사에 매핑되었음을 보여주는 서브에이전트 검증 표가 커밋 메시지에 포함됨.
- (6단계 이후) 살아있는 superpowers 문서 전반에서 삭제된 파일 경로에 대한 grep 결과가 0건임. `docs/superpowers/plans/*.md` 및 `RELEASE-NOTES.md`의 과거 참조는 재작성되지 않고 주석 처리됨.
- `docs/testing.md`에 "Plugin tests"와 "Skill behavior evals" 섹션이 모두 존재함.
- drill 저장소의 히스토리는 건드리지 않았으며, `obra/drill`은 이 PR의 영향을 받지 않음.
- PR 설명에 병합 후 `obra/drill`을 아카이브 처리하는 작업 항목이 명시됨.

## 미결 질문 (Open questions)

없음. 명확화가 필요한 모든 결정이 완료되었습니다:

| 질문 | 결정 사항 |
|---|---|
| superpowers 내에서 drill은 어디에 위치하는가? | `evals/` (drill에서 이름 변경); 독립 저장소는 별도 단계로 아카이브 처리 |
| 중복된 bash 테스트의 처리는? | 서브에이전트의 커버리지 검증을 거쳐 파일별로 삭제; 기본값은 유지(keep) |
| 시나리오 레이아웃은? | `evals/scenarios/`에 중앙 집중식으로 배치 |
| Python 툴체인 위치는? | `evals/` 내부에 자체 완결적(self-contained)으로 배치 |
| CI 통합은? | 이번 PR에서는 수동 실행만 지원; 향후 경로는 문서화함 |
| 마이그레이션 방식은? | 단순 복사; drill 저장소의 히스토리는 트리 내부가 아닌 아카이브된 저장소에 보존됨 |
| 내부 Python 패키지 이름은? | `drill`로 유지 (디렉터리 이름은 `evals/`) |
| 브랜치 전략은? | `dev`에서 독립적으로 분기 (`f/cross-platform` 위에 쌓지 않음) |
