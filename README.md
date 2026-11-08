# Superpowers

Superpowers는 코딩 에이전트를 위한 완전한 소프트웨어 개발 방법론으로, 조합 가능한(composable) 스킬(skills) 세트와 에이전트가 이를 확실히 사용하도록 보장하는 초기 지침(initial instructions) 위에 구축되었습니다.

## 목차 (Table of Contents)

- [작동 방식 (How it works)](#how-it-works)
- [상용 서비스 (Commercial Services)](#commercial-services)
- [시작하기 (Getting Started)](#installation)
  - [Claude Code](#claude-code)
  - [Antigravity](#antigravity)
  - [Codex App](#codex-app)
  - [Codex CLI](#codex-cli)
  - [Cursor](#cursor)
  - [Devin CLI](#devin-cli)
  - [Factory Droid](#factory-droid)
  - [Gemini CLI](#gemini-cli)
  - [GitHub Copilot CLI](#github-copilot-cli)
  - [Grok Build CLI](#grok-build-cli)
  - [Kimi Code](#kimi-code)
  - [OpenCode](#opencode)
  - [Pi](#pi)
  - [Qwen Code](#qwen-code)
  - [Hermes Agent](#hermes-agent)
  - [Muse](#muse)
- [기본 워크플로우 (The Basic Workflow)](#the-basic-workflow)
- [문제가 발생했을 때 (When Something Goes Wrong)](#when-something-goes-wrong)
- [커뮤니티 (Community)](#community)
- [포함된 내용 (What's Inside)](#whats-inside)
- [철학 (Philosophy)](#philosophy)
- [기여하기 (Contributing)](#contributing)
- [업데이트 (Updating)](#updating)
- [라이선스 (License)](#license)
- [시각적 컴패니언 텔레메트리 (Visual companion telemetry)](#visual-companion-telemetry)

<a id="how-it-works"></a>
## 작동 방식 (How it works)

코딩 에이전트를 실행하는 순간부터 시작됩니다. 에이전트는 여러분이 무언가를 만들고 있다는 것을 파악하더라도, 곧바로 코드를 작성하려고 덤벼들지 *않습니다*. 대신 한 걸음 물러서서 여러분이 진정으로 무엇을 하려는지 질문합니다.

대화를 통해 명세(spec)를 이끌어낸 후에는, 실제로 읽고 소화할 수 있을 만큼 짧은 단위(chunks)로 나누어 보여줍니다.

여러분이 설계(design)를 승인하면, 에이전트는 안목이 부족하고, 판단력이 없으며, 프로젝트 컨텍스트도 모르고, 테스트를 꺼리는 열정적인 주니어 엔지니어라도 그대로 따라 할 수 있을 만큼 명확한 구현 계획(implementation plan)을 작성합니다. 이 계획은 진정한 레드/그린 TDD(Red/Green TDD), YAGNI(You Aren't Gonna Need It), 그리고 DRY(Don't Repeat Yourself)를 강조합니다.

그다음 여러분이 "진행해(go)"라고 말하면, *서브에이전트 기반 개발(subagent-driven-development)* 프로세스를 시작하여 에이전트들이 각 엔지니어링 태스크를 수행하고, 작업물을 검사 및 리뷰한 뒤 다음 단계로 나아가도록 합니다. 에이전트가 여러분과 함께 세운 계획에서 벗어나지 않고 한 번에 두세 시간 동안 자율적으로 작업하는 일도 흔합니다.

이외에도 훨씬 더 많은 기능이 있지만, 이것이 시스템의 핵심입니다. 그리고 스킬들이 자동으로 트리거되기 때문에 여러분이 특별히 무언가를 할 필요가 없습니다. 여러분의 코딩 에이전트에게 그저 Superpowers가 생기는 것입니다.

<a id="commercial-services"></a>
## 상용 서비스 (Commercial Services)

엔터프라이즈 환경에서 Superpowers를 사용 중이며 상용 기술 지원, 추가 툴링 또는 비용 관리(managed spending)의 이점을 누리고 싶으시다면 언제든 sales@primeradiant.com으로 연락해 주세요.

<a id="installation"></a>
## 설치 (Installation)

설치 방법은 하네스(harness, 실행 환경/도구)에 따라 다릅니다. 둘 이상의 하네스를 사용한다면 각각에 대해 Superpowers를 별도로 설치하세요.

### Claude Code

Superpowers는 [공식 Claude 플러그인 마켓플레이스](https://claude.com/plugins/superpowers)를 통해 사용할 수 있습니다.

#### 공식 마켓플레이스 (Official Marketplace)

- Anthropic의 공식 마켓플레이스에서 플러그인을 설치합니다:

  ```bash
  /plugin install superpowers@claude-plugins-official
  ```

#### Superpowers 마켓플레이스 (Superpowers Marketplace)

Superpowers 마켓플레이스는 Claude Code를 위한 Superpowers 및 기타 관련 플러그인을 제공합니다.

- 마켓플레이스를 등록합니다:

  ```bash
  /plugin marketplace add obra/superpowers-marketplace
  ```

- 해당 마켓플레이스에서 플러그인을 설치합니다:

  ```bash
  /plugin install superpowers@superpowers-marketplace
  ```

### Antigravity

이 저장소에서 Superpowers를 플러그인으로 설치합니다:

```bash
agy plugin install https://github.com/obra/superpowers
```

Antigravity는 설치된 각 스킬의 설명(description)을 나열하며, `using-superpowers` 스킬의 설명은 대화가 시작될 때 모델이 해당 스킬을 로드하도록 유도합니다. 업데이트하려면 동일한 명령어로 재설치하세요.

### Codex App

Superpowers는 [공식 Codex 플러그인 마켓플레이스](https://github.com/openai/plugins)를 통해 사용할 수 있습니다.

- Codex 앱의 사이드바에서 Plugins를 클릭합니다.
- Coding 섹션에 `Superpowers`가 표시됩니다.
- Superpowers 옆의 `+`를 클릭하고 안내에 따릅니다.

### Codex CLI

Superpowers는 [공식 Codex 플러그인 마켓플레이스](https://github.com/openai/plugins)를 통해 사용할 수 있습니다.

- 플러그인 검색 인터페이스를 엽니다:

  ```bash
  /plugins
  ```

- Superpowers를 검색합니다:

  ```bash
  superpowers
  ```

- `Install Plugin`을 선택합니다.

### Cursor

- Cursor Agent 채팅창에서 마켓플레이스를 통해 설치합니다:

  ```text
  /add-plugin superpowers
  ```

- 또는 플러그인 마켓플레이스에서 "superpowers"를 검색합니다.

### Devin CLI

- 이 저장소에서 플러그인을 설치합니다:

  ```bash
  devin plugins install obra/superpowers
  ```

- 다음 명령어로 최신 버전으로 업데이트합니다:

  ```bash
  devin plugins update superpowers
  ```

### Factory Droid

- 마켓플레이스를 등록합니다:

  ```bash
  droid plugin marketplace add https://github.com/obra/superpowers
  ```

- 플러그인을 설치합니다:

  ```bash
  droid plugin install superpowers@superpowers
  ```

### Gemini CLI

- 익스텐션을 설치합니다:

  ```bash
  gemini extensions install https://github.com/obra/superpowers
  ```

- 추후 업데이트 방법:

  ```bash
  gemini extensions update superpowers
  ```

### GitHub Copilot CLI

- 마켓플레이스를 등록합니다:

  ```bash
  copilot plugin marketplace add obra/superpowers-marketplace
  ```

- 플러그인을 설치합니다:

  ```bash
  copilot plugin install superpowers@superpowers-marketplace
  ```

### Grok Build CLI

Superpowers는 [공식 Grok 플러그인 마켓플레이스](https://github.com/xai-org/plugin-marketplace)를 통해 사용할 수 있습니다.

- xAI의 공식 마켓플레이스에서 플러그인을 설치합니다:

  ```bash
  grok plugin install superpowers@xai-official --trust
  ```

- 또는 TUI에서 마켓플레이스를 열고 Superpowers를 검색하여 설치합니다:

  ```text
  /marketplace
  ```

### Kimi Code

Superpowers는 Kimi Code의 플러그인 마켓플레이스에서 사용할 수 있습니다.

- Kimi Code의 플러그인 매니저를 엽니다:

  ```text
  /plugins
  ```

- `Marketplace` > `Superpowers`로 이동하여 설치합니다.

- 또는 이 저장소에서 직접 설치합니다:

  ```text
  /plugins install https://github.com/obra/superpowers
  ```

- 상세 문서: [docs/README.kimi.md](docs/README.kimi.md)

### OpenCode

OpenCode는 자체 플러그인 설치 방식을 사용하므로, 다른 하네스에서 이미 사용 중이더라도 Superpowers를 별도로 설치해야 합니다.

- OpenCode에 다음과 같이 지시합니다:

  ```
  Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.opencode/INSTALL.md
  ```

- 상세 문서: [docs/README.opencode.md](docs/README.opencode.md)

### Pi

이 저장소에서 Superpowers를 Pi 패키지로 설치합니다:

```bash
pi install git:github.com/obra/superpowers
```

로컬 개발 시에는 이 체크아웃(checkout) 경로를 임시 패키지로 로드하여 Pi를 실행합니다:

```bash
pi -e /path/to/superpowers
```

Pi 패키지는 Superpowers 스킬들과 함께, 세션 시작 시점 및 컨텍스트 압축(compaction) 이후에 `using-superpowers` 부트스트랩(bootstrap)을 다시 주입하는 소규모 익스텐션을 로드합니다. Pi는 네이티브 스킬 기능을 갖추고 있으므로 별도의 호환성 `Skill` 도구가 필요하지 않습니다. 서브에이전트 및 태스크 목록 도구는 선택적인 Pi 컴패니언 패키지로 유지됩니다.

### Qwen Code

Qwen Code는 Claude Code 마켓플레이스에서 직접 플러그인을 설치합니다.

- 이 저장소에서 플러그인을 설치하고, 프롬프트가 표시되면 `superpowers`를 선택합니다:

  ```bash
  qwen extensions install obra/superpowers
  ```

- 추후 업데이트 방법:

  ```bash
  qwen extensions update superpowers
  ```

### Hermes Agent

이 저장소에서 Superpowers를 Hermes 플러그인으로 설치합니다:

```bash
hermes plugins install obra/superpowers --enable
```

설치 후 활성화된 모든 Hermes 세션을 재시작하세요. 참고: Hermes에는 압축 후(post-compaction) 훅이 없으므로, 세션이 매우 길어져 첫 번째 턴까지 압축되면 부트스트랩이 유실됩니다. 스킬이 더 이상 자동으로 트리거되지 않으면 새 세션을 시작하세요.

### Muse

Superpowers는 네이티브 Muse 플러그인으로 사용할 수 있습니다. 동일한 저장소와 동일한 스킬을 모든 하네스에서 공유합니다. `using-superpowers` 부트스트랩은 Claude Code, Codex, Cursor, Gemini, Pi 등과 마찬가지로 네이티브 `SessionStart` 훅을 통해 주입되므로 세션마다 별도로 활성화(opt-in)할 필요가 없습니다.

- 로컬 체크아웃에서 설치:

  ```bash
  muse plugins install ./
  muse plugins approve superpowers
  ```

  또는 클론 후 설치:

  ```bash
  git clone https://github.com/obra/superpowers.git
  muse plugins install ./superpowers
  muse plugins approve superpowers
  ```

- 추후 업데이트 방법:

  ```bash
  muse plugins update superpowers
  ```

설치 후 `SessionStart` 훅이 적용되도록 활성화된 모든 Muse 세션을 재시작하세요(스킬은 즉시 활성화되지만, 훅은 최초 설치 시 승인이 필요합니다). 정상 작동 여부를 확인하려면 새 세션을 시작하고 `Let's make a react todo list`를 보내보세요. 정상적으로 설치되었다면 코드를 작성하기 전에 `brainstorming` 스킬이 자동으로 트리거됩니다. 버전은 `.version-bump.json`에서 추적되므로 `scripts/bump-version.sh`가 이를 동기화 상태로 유지합니다.

<a id="the-basic-workflow"></a>
## 기본 워크플로우 (The Basic Workflow)

1. **brainstorming** - 무언가를 만들기 전에 활성화됩니다. 여러분이 실제로 원하는 것과 그 이유를 직접 설명하게 만드는 질문을 통해 의도를 이끌어낸 뒤, 자신이 이해한 내용을 다시 정리해 보여주며 교정을 받습니다. 작은 변경 사항은 채팅에서 합의하고, 프로젝트 단위의 작업은 구현자(builder)가 이를 바탕으로 계획을 세울 수 있는 설계 문서(design document)로 작성됩니다.

2. **using-git-worktrees** - 설계 승인 후 활성화됩니다. 새 브랜치에 격리된 작업 공간(isolated workspace)을 생성하고, 프로젝트 초기 설정을 실행하며, 기존 테스트가 깨끗하게 통과하는지(clean test baseline) 검증합니다.

3. **writing-plans** - 승인된 설계와 함께 활성화됩니다. 작업을 한 입 크기(bite-sized)의 태스크(각 2~5분 분량)로 쪼갭니다. 모든 태스크에는 정확한 파일 경로, 완전한 코드, 검증 단계가 포함됩니다.

4. **subagent-driven-development** 또는 **executing-plans** - 계획과 함께 활성화됩니다. 태스크마다 새로운 서브에이전트를 디스패치하고 각 태스크 직후 리뷰를 거치거나(가장 철저한 방식), 현재 세션 내에서 모든 태스크를 인라인으로 구현한 뒤 마지막에 전체 브랜치에 대해 한 번의 새로운 리뷰를 수행합니다(가장 비용 효율적인 방식).

5. **test-driven-development** - 구현 중에 활성화됩니다. RED-GREEN-REFACTOR 사이클을 강제합니다: 실패하는 테스트 작성, 실패 확인, 최소한의 코드 작성, 통과 확인, 커밋. 테스트보다 먼저 작성된 코드는 삭제합니다.

6. **requesting-code-review** - 태스크 사이에 활성화됩니다. 계획 대비 구현을 리뷰하고 심각도(severity)별로 이슈를 보고합니다. 치명적인(Critical) 이슈는 다음 진행을 차단합니다.

7. **finishing-a-development-branch** - 태스크가 완료되면 활성화됩니다. 테스트를 검증하고, 선택지(병합(merge) / PR 생성 / 유지(keep) / 폐기(discard))를 제시한 뒤, 워크트리(worktree)를 정리합니다.

**에이전트는 모든 태스크를 수행하기 전에 관련 스킬을 먼저 확인합니다.** 이는 권장 사항이 아니라 필수 워크플로우입니다.

<a id="when-something-goes-wrong"></a>
## 문제가 발생했을 때 (When Something Goes Wrong)

가끔 세션이 의도치 않게 동작할 때가 있습니다. 실행되지 말아야 할 때 스킬이 실행되거나, 실행되어야 할 때 침묵하거나, 에이전트가 계획을 무시하거나, 작업을 반복하거나, 예상보다 훨씬 많은 토큰을 소모하는 경우입니다. 코딩 에이전트에게 "이번 세션에서 superpowers에 무슨 문제가 있었는지 파악해 줘(figure out what went wrong with superpowers in this session)"라고 요청하면 **diagnosing-superpowers** 스킬을 호출합니다. 이전 세션을 조사하려면 세션 ID를 지정하세요: "세션 `<id>`에서 superpowers에 무슨 문제가 있었는지 파악해 줘(figure out what went wrong with superpowers in session `<id>`)"

이 스킬은 세션 트랜스크립트(transcript)를 읽고, 줄 단위의 근거와 함께 무슨 일이 일어났는지 보고하며, 원할 경우 버그 리포트용으로 민감 정보가 제거된(scrubbed) 번들을 패키징해 줍니다.

<a id="community"></a>
## 커뮤니티 (Community)

Superpowers는 [Jesse Vincent](https://blog.fsck.com)와 [Prime Radiant](https://primeradiant.com) 팀원들이 함께 만들고 있습니다.

- **Discord**: 커뮤니티 지원, 질문, 그리고 Superpowers로 만들고 있는 프로젝트를 공유하려면 [여기에 참여하세요](https://discord.gg/35wsABTejz)
- **Issues**: https://github.com/obra/superpowers/issues
- **릴리스 소식 (Release announcements)**: 새 버전 알림을 받으려면 [구독 신청하세요](https://primeradiant.com/superpowers/)

<a id="whats-inside"></a>
## 포함된 내용 (What's Inside)

### 스킬 라이브러리 (Skills Library)

**테스팅 (Testing)**
- **test-driven-development** - RED-GREEN-REFACTOR 사이클 (테스팅 안티패턴 레퍼런스 포함)

**디버깅 (Debugging)**
- **systematic-debugging** - 4단계 근본 원인(root cause) 분석 프로세스 (root-cause-tracing, defense-in-depth, condition-based-waiting 기법 포함)
- **verification-before-completion** - 문제가 실제로 해결되었는지 확인
- **diagnosing-superpowers** - 세션에서 무엇이 잘못되었는지 근거와 함께 파악하고, 민감 정보가 제거된 번들을 내보내거나 이슈를 등록

**협업 (Collaboration)** 
- **brainstorming** - 구현 전 의도 이끌어내기
- **writing-plans** - 상세한 구현 계획 작성
- **executing-plans** - 인라인 계획 실행: 단일 컨텍스트, 마지막에 한 번의 최종 리뷰
- **dispatching-parallel-agents** - 동시 실행 서브에이전트 워크플로우
- **requesting-code-review** - 리뷰 전 체크리스트
- **receiving-code-review** - 피드백에 대응하기
- **using-git-worktrees** - 병렬 개발 브랜치 운용
- **finishing-a-development-branch** - 병합(Merge)/PR 의사결정 워크플로우
- **subagent-driven-development** - 2단계 리뷰(명세 준수 여부 확인 후 코드 품질 확인)를 통한 빠른 이터레이션

**메타 (Meta)**
- **writing-skills** - 모범 사례에 따라 새로운 스킬 생성 (테스팅 방법론 포함)
- **using-superpowers** - 스킬 시스템 입문 및 안내

<a id="philosophy"></a>
## 철학 (Philosophy)

- **테스트 주도 개발 (Test-Driven Development)** - 언제나 테스트를 먼저 작성합니다
- **임기응변보다 체계적인 접근 (Systematic over ad-hoc)** - 추측보다 프로세스를 따릅니다
- **복잡성 감소 (Complexity reduction)** - 단순함을 최우선 목표로 삼습니다
- **주장보다 증거 (Evidence over claims)** - 성공을 선언하기 전에 검증합니다

[최초 릴리스 발표 글](https://blog.fsck.com/2025/10/09/superpowers/)을 읽어보세요.

<a id="contributing"></a>
## 기여하기 (Contributing)

Superpowers의 일반적인 기여 프로세스는 다음과 같습니다. 참고로 새로운 스킬을 추가하는 기여는 일반적으로 받지 않으며, 기존 스킬에 대한 모든 업데이트는 우리가 지원하는 모든 코딩 에이전트에서 작동해야 합니다. 또한 다른 사람이 등록한 이슈에 대해 메인테이너가 요청하지 않은 한, 설령 여러분이 동일한 문제를 겪었더라도 에이전트가 작성한 수정 PR은 받지 않습니다. 대신 기존 이슈에 여러분의 사례 리포트를 댓글로 추가해 주세요.

1. 저장소를 포크(Fork)합니다.
2. 'dev' 브랜치로 전환합니다.
3. 작업을 위한 새 브랜치를 생성합니다.
4. 신규 및 수정된 스킬을 작성하고 테스트할 때는 `writing-skills` 스킬을 따릅니다.
5. 인간이 직접 자신의 말로 작성한 요약을 포함하여, 풀 리퀘스트 템플릿의 모든 섹션을 빠짐없이 채워 PR을 제출합니다.

스킬 동작 테스트는 `evals/`에 클론된 [superpowers-evals](https://github.com/prime-radiant-inc/superpowers-evals/)의 drill 평가 하네스(eval harness)를 사용합니다(설정 방법은 `evals/README.md` 참조). 플러그인 인프라 테스트는 `tests/`에 위치하며 관련 `run-*.sh` 또는 `npm test`를 통해 실행됩니다.

전체 가이드는 `skills/writing-skills/SKILL.md`를 참조하세요.

<a id="updating"></a>
## 업데이트 (Updating)

Superpowers 업데이트 방식은 사용하는 코딩 에이전트에 따라 다소 차이가 있지만, 대개 자동으로 이루어집니다.

<a id="license"></a>
## 라이선스 (License)

MIT License - 자세한 내용은 LICENSE 파일을 참조하세요.

<a id="visual-companion-telemetry"></a>
## 시각적 컴패니언 텔레메트리 (Visual companion telemetry)

스킬과 플러그인은 제작자에게 어떠한 피드백도 제공하지 않기 때문에, 우리는 얼마나 많은 분들이 Superpowers를 사용하고 있는지 알 수 없습니다. 기본적으로 `brainstorming`의 선택적 기능인 시각적 컴패니언(visual companion)에 표시되는 Prime Radiant 로고는 우리 웹사이트에서 로드됩니다. 이때 사용 중인 Superpowers 버전 정보만 함께 전달됩니다. 여러분의 프로젝트, 프롬프트, 또는 코딩 에이전트에 대한 어떠한 세부 정보도 포함되지 않습니다. 우리는 여러분이 무엇을 클릭하는지나 무엇을 만들고 있는지 전혀 볼 수 없습니다. 이는 단지 얼마나 많은 분들이 어떤 버전의 Superpowers를 사용하고 있는지 대략적으로 파악하는 데만 도움을 줍니다. 이 기능은 100% 선택 사항입니다. 비활성화하려면 환경 변수 `SUPERPOWERS_DISABLE_TELEMETRY`를 임의의 참(true) 값으로 설정하세요. 또한 Superpowers는 Claude Code의 `DISABLE_TELEMETRY` 및 `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` 옵트아웃(opt-out) 설정도 존중합니다.
