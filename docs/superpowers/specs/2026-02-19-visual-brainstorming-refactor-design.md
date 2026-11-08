# 시각적 브레인스토밍 리팩토링: 브라우저 표시, 터미널 명령 (Visual Brainstorming Refactor: Browser Displays, Terminal Commands)

**날짜:** 2026-02-19
**상태:** 승인됨 (Approved)
**범위:** `lib/brainstorm-server/`, `skills/brainstorming/visual-companion.md`, `tests/brainstorm-server/`

## 문제점

시각적 브레인스토밍 중에 Claude는 `wait-for-feedback.sh`를 백그라운드 태스크로 실행하고 `TaskOutput(block=true, timeout=600s)`에서 블로킹됩니다. 이로 인해 TUI가 완전히 점유되어 시각적 브레인스토밍이 실행되는 동안 사용자가 Claude에 타이핑을 할 수 없습니다. 브라우저가 유일한 입력 채널이 되어버립니다.

Claude Code의 실행 모델은 턴 기반(turn-based)입니다. 단일 턴 내에서 Claude가 두 개의 채널을 동시에 리스닝할 수 있는 방법은 없습니다. 블로킹 `TaskOutput` 패턴은 잘못된 프리미티브였습니다 — 이는 플랫폼이 지원하지 않는 이벤트 기반 동작을 흉내 내려 한 것입니다.

## 설계

### 핵심 모델

**브라우저 = 인터랙티브 디스플레이.** 목업을 보여주고 사용자가 클릭하여 옵션을 선택할 수 있게 합니다. 선택 사항은 서버 측에 기록됩니다.

**터미널 = 대화 채널.** 항상 블로킹되지 않고 언제나 사용할 수 있습니다. 사용자는 여기서 Claude와 대화합니다.

### 루프 (The Loop)

1. Claude가 세션 디렉터리에 HTML 파일을 작성합니다.
2. 서버가 chokidar를 통해 이를 감지하고 브라우저에 WebSocket 리로드를 푸시합니다 (변경 없음).
3. Claude가 자신의 턴을 종료합니다 — 사용자에게 브라우저를 확인하고 터미널에서 응답해 달라고 안내합니다.
4. 사용자가 브라우저를 보고, 선택적으로 클릭하여 옵션을 고른 뒤, 터미널에 피드백을 입력합니다.
5. 다음 턴에서 Claude는 `$SCREEN_DIR/.events`를 읽어 브라우저 상호작용 스트림(클릭, 선택)을 확인하고 이를 터미널 텍스트와 병합합니다.
6. 반복하거나 다음 단계로 넘어갑니다.

백그라운드 태스크가 없습니다. `TaskOutput` 블로킹도 없습니다. 폴링 스크립트도 없습니다.

### 주요 삭제 사항: `wait-for-feedback.sh`

완전히 삭제됩니다. 이 스크립트의 목적은 "서버가 이벤트를 stdout에 로깅함"과 "Claude가 해당 이벤트를 수신해야 함" 사이를 연결하는 것이었습니다. `.events` 파일이 이를 대체합니다 — 서버는 사용자 상호작용 이벤트를 직접 파일에 기록하고, Claude는 플랫폼이 제공하는 파일 읽기 메커니즘을 사용하여 이를 읽습니다.

### 주요 추가 사항: `.events` 파일 (화면별 이벤트 스트림)

서버는 모든 사용자 상호작용 이벤트를 한 줄에 하나의 JSON 객체 형식으로 `$SCREEN_DIR/.events`에 기록합니다. 이를 통해 Claude는 최종 선택뿐만 아니라 사용자의 탐색 경로(A를 클릭했다가 B를 클릭하고 최종적으로 C로 결정함)를 포함한 현재 화면의 전체 상호작용 스트림을 파악할 수 있습니다.

사용자가 옵션을 탐색한 후의 파일 내용 예시:

```jsonl
{"type":"click","choice":"a","text":"Option A - Preset-First Wizard","timestamp":1706000101}
{"type":"click","choice":"c","text":"Option C - Manual Config","timestamp":1706000108}
{"type":"click","choice":"b","text":"Option B - Hybrid Approach","timestamp":1706000115}
```

- 단일 화면 내에서는 추가 전용(append-only)으로 동작합니다. 각 사용자 이벤트가 새로운 줄로 추가됩니다.
- chokidar가 새로운 HTML 파일(새 화면 푸시)을 감지하면 파일이 초기화(삭제)되어 이전 화면의 오래된 이벤트가 넘어오는 것을 방지합니다.
- Claude가 읽으려 할 때 파일이 존재하지 않는다면 브라우저 상호작용이 없었던 것입니다 — Claude는 터미널 텍스트만 사용합니다.
- 이 파일에는 서버 수명주기 이벤트(`server-started`, `screen-added`)가 아닌 사용자 이벤트(`click` 등)만 포함됩니다. 이를 통해 파일을 작고 목적에 맞게 유지합니다.
- Claude는 전체 스트림을 읽어 사용자의 탐색 패턴을 이해하거나, 단순히 마지막 `choice` 이벤트만 확인하여 최종 선택을 파악할 수 있습니다.

## 파일별 변경 사항

### `index.js` (서버)

**A. 사용자 이벤트를 `.events` 파일에 기록.**

WebSocket `message` 핸들러에서 이벤트를 stdout에 로깅한 후: `fs.appendFileSync`를 통해 `$SCREEN_DIR/.events`에 이벤트를 JSON 줄로 추가합니다. 서버 수명주기 이벤트가 아닌 사용자 상호작용 이벤트(`source: 'user-event'`가 있는 이벤트)만 기록합니다.

**B. 새 화면에서 `.events` 초기화.**

chokidar `add` 핸들러(새로운 `.html` 파일 감지)에서 `$SCREEN_DIR/.events`가 존재하면 삭제합니다. 이는 리로드할 때마다 발생하는 GET `/`에서 초기화하는 것보다 훨씬 명확한 "새 화면" 신호입니다.

**C. `wrapInFrame` 콘텐츠 주입 방식 교체.**

현재 정규식은 삭제 예정인 `<div class="feedback-footer">`를 기준으로 삼고 있습니다. 이를 주석 플레이스홀더로 교체합니다: `#claude-content` 내부의 기존 기본 콘텐츠(`<h2>Visual Brainstorming</h2>` 및 부제목 단락)를 제거하고 단일 `<!-- CONTENT -->` 마커로 대체합니다. 콘텐츠 주입은 `frameTemplate.replace('<!-- CONTENT -->', content)`가 됩니다. 더 단순해지며 템플릿 포맷이 변경되더라도 깨지지 않습니다.

### `frame-template.html` (UI 프레임)

**제거:**
- `feedback-footer` div (textarea, Send 버튼, label, `.feedback-row`)
- 관련 CSS (`.feedback-footer`, `.feedback-footer label`, `.feedback-row`, 내부의 textarea 및 버튼 스타일)

**추가:**
- 기본 텍스트를 대체하여 `#claude-content` 내부에 `<!-- CONTENT -->` 플레이스홀더 추가
- 기존 푸터 자리에 두 가지 상태를 가지는 선택 표시줄(selection indicator bar) 추가:
  - 기본 상태: "Click an option above, then return to the terminal"
  - 선택 후 상태: "Option B selected — return to terminal to continue"
- 표시줄을 위한 CSS (기존 헤더와 유사한 시각적 무게감을 가진 은은한 스타일)

**그대로 유지:**
- "Brainstorm Companion" 제목과 연결 상태가 있는 헤더 바
- `.main` 래퍼 및 `#claude-content` 컨테이너
- 모든 컴포넌트 CSS (`.options`, `.cards`, `.mockup`, `.split`, `.pros-cons`, 플레이스홀더, 목업 요소들)
- 다크/라이트 테마 변수 및 미디어 쿼리

### `helper.js` (클라이언트 측 스크립트)

**제거:**
- `sendToClaude()` 함수 및 "Sent to Claude" 화면 전환 오버레이
- `window.send()` 함수 (제거된 Send 버튼에 연결되어 있었음)
- 폼 제출 핸들러 — 피드백 textarea가 없으므로 목적이 없고 로그 노이즈만 유발함
- 입력 변경(input change) 핸들러 — 동일한 이유
- `pageshow` 이벤트 리스너 (textarea 내용이 남아있는 문제를 수정하기 위해 추가되었으나 이제 textarea가 없음)

**유지:**
- WebSocket 연결, 재연결 로직, 이벤트 큐
- 리로드 핸들러 (서버 푸시 시 `window.location.reload()`)
- 선택 강조 표시를 위한 `window.toggleSelect()`
- `window.selectedChoice` 추적
- `window.brainstorm.send()` 및 `window.brainstorm.choice()` — 이는 제거된 `window.send()`와는 별개입니다. 이들은 WebSocket을 통해 서버에 로깅하는 `sendEvent`를 호출합니다. 커스텀 전체 문서(full-document) 페이지에 유용합니다.

**축소:**
- 클릭 핸들러: 모든 버튼/링크가 아니라 `[data-choice]` 클릭만 캡처합니다. 브라우저가 피드백 채널이었을 때는 광범위한 캡처가 필요했지만, 이제는 선택 추적 용도로만 사용됩니다.

**추가:**
- `data-choice` 클릭 시, 어떤 옵션이 선택되었는지 보여주도록 선택 표시줄의 텍스트를 업데이트합니다.

**`window.brainstorm` API에서 제거:**
- `brainstorm.sendToClaude` — 더 이상 존재하지 않음

### `visual-companion.md` (스킬 지침)

**"The Loop" 섹션 재작성:** 위에서 설명한 논블로킹 흐름으로 다시 작성합니다. 다음 참조를 모두 제거합니다:
- `wait-for-feedback.sh`
- `TaskOutput` 블로킹
- 타임아웃/재시도 로직 (600초 타임아웃, 30분 제한)
- `send-to-claude` JSON을 설명하는 "User Feedback Format" 섹션

**다음 내용으로 대체:**
- 새로운 루프 (HTML 작성 → 턴 종료 → 사용자가 터미널에서 응답 → `.events` 읽기 → 반복)
- `.events` 파일 포맷 문서화
- 터미널 메시지가 주된 피드백이며, `.events`는 추가 컨텍스트를 위해 전체 브라우저 상호작용 스트림을 제공한다는 지침

**유지:**
- 서버 시작/종료 지침
- 콘텐츠 프래그먼트(fragment) 대 전체 문서(full document) 가이드라인
- CSS 클래스 레퍼런스 및 사용 가능한 컴포넌트
- 디자인 팁 (질문에 맞춘 충실도 조절, 화면당 2~4개 옵션 등)

### `wait-for-feedback.sh`

**완전히 삭제됨.**

### `tests/brainstorm-server/server.test.js`

업데이트가 필요한 테스트:
- 프래그먼트 응답에서 `feedback-footer`의 존재를 검증하는 테스트 — 선택 표시줄 또는 `<!-- CONTENT -->` 치환을 검증하도록 업데이트
- `helper.js`에 `send`가 포함되어 있는지 검증하는 테스트 — 축소된 API를 반영하도록 업데이트
- `sendToClaude` CSS 변수 사용을 검증하는 테스트 — 제거 (해당 함수가 더 이상 존재하지 않음)

## 플랫폼 호환성

서버 코드(`index.js`, `helper.js`, `frame-template.html`)는 플랫폼에 전혀 종속되지 않습니다 — 순수 Node.js와 브라우저 JavaScript로만 작성되었습니다. Claude Code 전용 참조가 없으며, 백그라운드 터미널 상호작용을 통해 Codex에서도 작동함이 이미 입증되었습니다.

스킬 지침(`visual-companion.md`)은 플랫폼 적응형 계층입니다. 각 플랫폼의 에이전트는 자체 도구를 사용하여 서버를 시작하고 `.events`를 읽는 등의 작업을 수행합니다. 논블로킹 모델은 플랫폼별 블로킹 프리미티브에 의존하지 않으므로 여러 플랫폼에서 자연스럽게 작동합니다.

## 이 설계가 가능하게 하는 것

- 시각적 브레인스토밍 중에도 **TUI가 항상 응답 가능함**
- **혼합 입력** — 브라우저에서의 클릭 + 터미널에서의 타이핑이 자연스럽게 병합됨
- **우아한 성능 저하 (Graceful degradation)** — 브라우저가 내려가 있거나 사용자가 열지 않더라도 터미널은 여전히 작동함
- **더 단순해진 아키텍처** — 백그라운드 태스크, 폴링 스크립트, 타임아웃 관리가 필요 없음
- **크로스 플랫폼** — 동일한 서버 코드가 Claude Code, Codex 및 향후 모든 플랫폼에서 작동함

## 포기하는 것 (트레이드오프)

- **순수 브라우저 기반 피드백 워크플로우** — 사용자는 계속 진행하려면 반드시 터미널로 돌아와야 합니다. 선택 표시줄이 이를 안내해 주지만, 기존의 '클릭 후 Send 누르고 대기'하는 흐름에 비하면 한 단계가 추가됩니다.
- **브라우저 내 인라인 텍스트 피드백** — textarea가 사라졌습니다. 모든 텍스트 피드백은 터미널을 통해 이루어집니다. 이는 의도된 설계입니다 — 프레임 안의 작은 textarea보다 터미널이 훨씬 더 나은 텍스트 입력 채널이기 때문입니다.
- **브라우저 Send 클릭 시 즉각적인 응답** — 기존 시스템에서는 사용자가 Send를 클릭하는 순간 Claude가 응답했습니다. 이제는 사용자가 터미널로 전환하는 동안 약간의 간격이 생깁니다. 실제로는 몇 초에 불과하며, 사용자는 터미널 메시지에 추가 컨텍스트를 덧붙일 수 있습니다.
