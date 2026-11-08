# 시각적 컴패니언 가이드 (Visual Companion Guide)

목업(mockups), 다이어그램, 선택지들을 보여주기 위한 브라우저 기반의 시각적 브레인스토밍 컴패니언입니다.

## 언제 사용할 것인가 (When to Use)

세션 단위가 아니라 질문 단위로 결정하세요. 기준은 다음과 같습니다: **사용자가 이것을 읽는 것보다 눈으로 볼 때 더 잘 이해할까?**

콘텐츠 자체가 시각적일 때는 **브라우저를 사용하세요**:

- **UI 목업 (UI mockups)** — 와이어프레임, 레이아웃, 내비게이션 구조, 컴포넌트 디자인
- **아키텍처 다이어그램 (Architecture diagrams)** — 시스템 컴포넌트, 데이터 흐름, 관계도
- **나란히 놓고 보는 시각적 비교 (Side-by-side visual comparisons)** — 두 가지 레이아웃, 두 가지 색상 테마, 두 가지 디자인 방향성 비교
- **디자인 다듬기 (Design polish)** — 룩앤필(look and feel), 여백(spacing), 시각적 계층 구조에 관한 질문일 때
- **공간적 관계 (Spatial relationships)** — 다이어그램으로 렌더링된 상태 머신(state machines), 순서도(flowcharts), 엔티티 관계(ER)

콘텐츠가 텍스트이거나 표 형태일 때는 **터미널을 사용하세요**:

- **요구사항 및 범위 질문 (Requirements and scope questions)** — "X는 무슨 뜻인가요?", "어떤 기능들이 범위에 포함되나요?"
- **개념적인 A/B/C 선택 (Conceptual A/B/C choices)** — 말로 설명된 접근 방식들 중에서 선택하기
- **트레이드오프 목록 (Tradeoff lists)** — 장단점(pros/cons), 비교 표
- **기술적 의사결정 (Technical decisions)** — API 설계, 데이터 모델링, 아키텍처 접근 방식 선택
- **명확화 질문 (Clarifying questions)** — 시각적 선호도가 아니라 말(단어)로 답해야 하는 모든 질문

UI 주제에 *관한* 질문이라고 해서 자동으로 시각적 질문이 되는 것은 아닙니다. "어떤 종류의 마법사(wizard) UI를 원하시나요?"는 개념적 질문이므로 터미널을 사용하세요. "이 마법사 레이아웃들 중 어느 것이 가장 자연스럽게 느껴지시나요?"는 시각적 질문이므로 브라우저를 사용하세요.

## 작동 방식 (How It Works)

서버는 특정 디렉터리의 HTML 파일들을 감시(watch)하다가 가장 최신 파일을 브라우저에 서빙합니다. 당신이 `screen_dir`에 HTML 콘텐츠를 작성하면, 사용자는 브라우저에서 이를 확인하고 클릭하여 옵션을 선택할 수 있습니다. 선택 내역은 `state_dir/events`에 기록되며 당신은 다음 턴에 이를 읽어 들입니다.

**콘텐츠 프래그먼트 vs 전체 문서 (Content fragments vs full documents):** 작성한 HTML 파일이 `<!DOCTYPE` 또는 `<html`로 시작하면 서버는 해당 파일을 있는 그대로 서빙합니다(헬퍼 스크립트만 주입). 그렇지 않으면 서버는 자동으로 콘텐츠를 프레임 템플릿으로 감싸서 헤더, CSS 테마, 연결 상태 표시기 및 모든 인터랙티브 인프라를 추가합니다. **기본적으로 콘텐츠 프래그먼트(content fragments)를 작성하세요.** 페이지 전체를 완전히 제어해야 할 때만 전체 문서를 작성하세요.

## 세션 시작하기 (Starting a Session)

```bash
# 사용자가 컴패니언 사용을 승인한 후에 시작하세요. --open은 첫 번째 화면이 준비되면
# 사용자의 브라우저를 자동으로 엽니다. --project-dir은 목업을 영구 보관하고 동일 포트 재시작을 활성화합니다.
bash scripts/start-server.sh --project-dir /path/to/project --open

# 반환값 예시: {"type":"server-started","port":52341,
#           "url":"http://localhost:52341/?key=ab12…",
#           "screen_dir":"/path/to/project/.superpowers/brainstorm/12345-1706000000/content",
#           "state_dir":"/path/to/project/.superpowers/brainstorm/12345-1706000000/state"}
```

응답에서 `screen_dir`과 `state_dir`을 저장해 두세요. `--open` 옵션을 사용하면 첫 번째 화면을 푸시할 때 브라우저가 스스로 열리므로 사용자에게 직접 열어달라고 요청할 필요는 없지만, 헤드리스(headless)나 원격 환경에서는 자동으로 열리지 않으므로 대비책(fallback)으로 URL은 항상 함께 공유하세요.

**URL에는 세션 키(`?key=…`)가 포함되어 있습니다.** 서버는 이 키가 없는 모든 요청을 거부하므로, 사용자에게는 항상 `url` 필드에 있는 **완전한** URL을 전달하세요. 절대 쿼리 스트링을 잘라내거나 단순한 `http://host:port`만 건네주지 마세요. 이 키는 HTTP 및 WebSocket 접근을 제어하여, 엉뚱한 브라우저 탭이나 네트워크상의 다른 머신이 화면을 읽거나 이벤트를 주입하지 못하도록 보호합니다. 첫 로드 이후에는 브라우저가 쿠키를 통해 키를 기억하므로 페이지 새로고침이나 `/files/*` 에셋 로드는 키를 반복해서 붙이지 않아도 작동합니다.

**연결 정보 찾기 (Finding connection info):** 서버는 시작 JSON 정보를 `$STATE_DIR/server-info`에 기록합니다. 백그라운드에서 서버를 실행하여 표준 출력(stdout)을 캡처하지 못했다면 해당 파일을 읽어 URL과 포트를 확인하세요. `--project-dir`을 사용하는 경우 `<project>/.superpowers/brainstorm/`에서 세션 디렉터리를 확인할 수 있습니다.

**참고:** 목업이 `.superpowers/brainstorm/`에 영구 보존되어 서버 재시작 후에도 유지되도록 프로젝트 루트를 `--project-dir`로 전달하세요. 이 옵션이 없으면 파일들이 `/tmp`에 저장되어 추후 정리(삭제)됩니다. `.gitignore`에 `.superpowers/`가 아직 없다면 추가하도록 사용자에게 안내하세요.

**플랫폼별 서버 실행 방법 (Launching the server by platform):**

**Claude Code:**
```bash
# 기본 모드가 잘 작동합니다 — 스크립트 자체적으로 서버를 백그라운드로 전환합니다.
bash scripts/start-server.sh --project-dir /path/to/project --open
```

Windows에서는 스크립트가 환경을 자동 감지하여 포그라운드(foreground) 모드로 전환합니다(도구 호출이 블로킹됨). 대화 턴이 넘어가도 서버가 살아남도록 Bash 도구 호출에 `run_in_background: true`를 사용하고, 다음 턴에 `$STATE_DIR/server-info`를 읽어 URL과 포트를 가져오세요.

**Codex:**
```bash
# Codex는 백그라운드 프로세스를 종료(reap)시킵니다. 스크립트가 CODEX_CI를 자동 감지하여
# 포그라운드 모드로 전환하므로 별도 플래그 없이 평소처럼 실행하면 됩니다.
bash scripts/start-server.sh --project-dir /path/to/project --open
```

**Gemini CLI:**
```bash
# 프로세스가 턴을 넘어 유지되도록 --foreground를 사용하고
# 셸 도구 호출에서 is_background: true를 설정하세요.
bash scripts/start-server.sh --project-dir /path/to/project --open --foreground
```

**Copilot CLI:**
```bash
# 서버가 턴을 넘어 유지되도록 Copilot CLI의 논블로킹/백그라운드 셸 메커니즘으로 시작하세요.
# 스크립트가 아닌 하네스가 백그라운드 관리를 소유하도록 --foreground를 유지합니다.
# 런처는 .sh 파일이므로 bash를 통해 호출하세요 (Windows에서는 PowerShell 도구에서 Git Bash의 bash.exe 호출).
bash scripts/start-server.sh --project-dir /path/to/project --open --foreground
```

**기타 환경 (Other environments):** 서버는 대화 턴이 넘어가는 동안에도 백그라운드에서 계속 실행 중이어야 합니다. 만약 실행 환경이 분리된(detached) 프로세스를 강제 종료시킨다면 `--foreground`를 사용하고 해당 플랫폼의 백그라운드 실행 메커니즘으로 명령을 실행하세요.

브라우저에서 URL에 접근할 수 없는 경우(원격 또는 컨테이너 환경에서 흔함), 논루프백(non-loopback) 호스트를 바인딩하세요:

```bash
bash scripts/start-server.sh \
  --project-dir /path/to/project \
  --host 0.0.0.0 \
  --url-host localhost
```

반환되는 URL JSON에 출력될 호스트네임을 제어하려면 `--url-host`를 사용하세요.

## 반복 루프 (The Loop)

1. **서버가 살아있는지 확인한 후**, `screen_dir`의 새 파일에 **HTML을 작성합니다**:
   - **필수: URL을 언급하거나 화면을 푸시하기 전에 서버가 살아있는지 반드시 확인하세요.** `$STATE_DIR/server-info`가 존재하고 `$STATE_DIR/server-stopped`가 존재하지 않는지 확인합니다. 서버가 종료되었다면 **동일한 `--project-dir`**을 사용하여 `start-server.sh`로 재시작하세요. 동일한 포트를 재사용하므로 사용자가 열어둔 탭이 스스로 재연결되며(서버가 내려가 있는 동안에는 "일시 중지됨(paused)" 오버레이가 표시됨) 새 URL을 보낼 필요가 없습니다. 서버는 유휴 상태가 4시간 지속되면 자동 종료됩니다(`--idle-timeout-minutes`로 설정 가능).
   - 의미 있는(semantic) 파일 이름을 사용하세요: `platform.html`, `visual-style.html`, `layout.html`
   - **절대 파일 이름을 재사용하지 마세요** — 각 화면은 새로운 파일이어야 합니다.
   - 파일 생성 도구를 사용하세요 — **절대 cat/heredoc을 사용하지 마세요** (터미널에 불필요한 노이즈를 쏟아냅니다).
   - 서버는 가장 최신 파일을 자동으로 서빙합니다.

2. **사용자에게 무엇을 보게 될지 알리고 턴을 종료합니다:**
   - URL을 다시 알려주세요 (첫 단계뿐만 아니라 매 단계마다).
   - 화면에 무엇이 표시되어 있는지 간략한 텍스트 요약을 제공하세요 (예: "홈페이지를 위한 3가지 레이아웃 옵션을 보여드리고 있습니다").
   - 터미널에서 응답해 달라고 요청하세요: "확인해 보시고 어떻게 생각하시는지 알려주세요. 원하시면 클릭해서 옵션을 선택하실 수 있습니다."

3. **다음 턴에서** — 사용자가 터미널에 응답한 후:
   - `$STATE_DIR/events` 파일이 존재한다면 읽으세요 — 여기에는 사용자의 브라우저 인터랙션(클릭, 선택)이 JSON 라인(JSONL) 형태로 담겨 있습니다.
   - 사용자의 터미널 텍스트 응답과 병합하여 전체 맥락을 파악하세요.
   - 터미널 메시지가 주된 피드백이며, `state_dir/events`는 구조화된 인터랙션 데이터를 제공합니다.

4. **반복(Iterate)하거나 다음으로 전진(Advance)** — 피드백으로 인해 현재 화면이 수정되어야 한다면 새 파일을 작성하세요(예: `layout-v2.html`). 현재 단계가 검증되었을 때만 다음 질문으로 넘어가세요.

5. **터미널 대화로 돌아갈 때는 화면 비우기(Unload)** — 다음 단계에서 브라우저가 필요하지 않을 때(예: 명확화 질문, 트레이드오프 논의)는 이전의 오래된 콘텐츠를 지우기 위해 대기 화면(waiting screen)을 푸시하세요:

   ```html
   <!-- 파일명: waiting.html (또는 waiting-2.html 등) -->
   <div style="display:flex;align-items:center;justify-content:center;min-height:60vh">
     <p class="subtitle">Continuing in terminal...</p>
   </div>
   ```

   이렇게 하면 대화가 이미 다음 주제로 넘어갔는데도 사용자가 이미 결정 끝난 선택지 화면을 계속 쳐다보고 있는 상황을 방지할 수 있습니다. 다음 시각적 질문이 나오면 평소처럼 새로운 콘텐츠 파일을 푸시하세요.

6. 완료될 때까지 반복합니다.

## 콘텐츠 프래그먼트 작성하기 (Writing Content Fragments)

페이지 내부에 들어갈 콘텐츠만 작성하세요. 서버가 이를 자동으로 프레임 템플릿(헤더, 테마 CSS, 연결 상태 표시기 및 모든 인터랙티브 인프라)으로 감싸줍니다.

**최소 예제 (Minimal example):**

```html
<h2>어느 레이아웃이 더 나은가요?</h2>
<p class="subtitle">가독성과 시각적 계층 구조를 고려해 보세요</p>

<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>단일 컬럼 (Single Column)</h3>
      <p>깔끔하고 집중도 높은 읽기 경험</p>
    </div>
  </div>
  <div class="option" data-choice="b" onclick="toggleSelect(this)">
    <div class="letter">B</div>
    <div class="content">
      <h3>2단 컬럼 (Two Column)</h3>
      <p>사이드바 내비게이션과 메인 콘텐츠 구성</p>
    </div>
  </div>
</div>
```

이것으로 충분합니다. `<html>`, CSS, `<script>` 태그는 필요하지 않습니다. 서버가 그 모든 것을 제공합니다.

## 사용 가능한 CSS 클래스 (CSS Classes Available)

프레임 템플릿은 콘텐츠를 위해 다음과 같은 CSS 클래스들을 제공합니다:

### 옵션 (Options — A/B/C 선택지)

```html
<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>제목</h3>
      <p>설명</p>
    </div>
  </div>
</div>
```

**다중 선택 (Multi-select):** 사용자가 여러 옵션을 선택할 수 있게 하려면 컨테이너에 `data-multiselect`를 추가하세요. 클릭할 때마다 해당 항목의 선택 스타일이 토글됩니다.

```html
<div class="options" data-multiselect>
  <!-- 동일한 옵션 마크업 — 사용자가 여러 개를 선택/해제할 수 있음 -->
</div>
```

### 카드 (Cards — 시각적 디자인)

```html
<div class="cards">
  <div class="card" data-choice="design1" onclick="toggleSelect(this)">
    <div class="card-image"><!-- 목업 콘텐츠 --></div>
    <div class="card-body">
      <h3>이름</h3>
      <p>설명</p>
    </div>
  </div>
</div>
```

### 목업 컨테이너 (Mockup container)

```html
<div class="mockup">
  <div class="mockup-header">Preview: Dashboard Layout</div>
  <div class="mockup-body"><!-- 목업 HTML --></div>
</div>
```

### 분할 뷰 (Split view — 나란히 비교)

```html
<div class="split">
  <div class="mockup"><!-- 왼쪽 --></div>
  <div class="mockup"><!-- 오른쪽 --></div>
</div>
```

### 장단점 (Pros/Cons)

```html
<div class="pros-cons">
  <div class="pros"><h4>Pros</h4><ul><li>장점</li></ul></div>
  <div class="cons"><h4>Cons</h4><ul><li>단점</li></ul></div>
</div>
```

### 목(Mock) 요소 (와이어프레임 빌딩 블록)

```html
<div class="mock-nav">Logo | Home | About | Contact</div>
<div style="display: flex;">
  <div class="mock-sidebar">Navigation</div>
  <div class="mock-content">Main content area</div>
</div>
<button class="mock-button">Action Button</button>
<input class="mock-input" placeholder="Input field">
<div class="placeholder">Placeholder area</div>
```

### 타이포그래피 및 섹션 (Typography and sections)

- `h2` — 페이지 제목
- `h3` — 섹션 제목
- `.subtitle` — 제목 아래의 보조 텍스트
- `.section` — 하단 여백(bottom margin)이 있는 콘텐츠 블록
- `.label` — 작은 대문자 라벨 텍스트

## 브라우저 이벤트 포맷 (Browser Events Format)

사용자가 브라우저에서 옵션을 클릭하면, 그 인터랙션이 `$STATE_DIR/events`에 한 줄당 하나의 JSON 객체 형태로 기록됩니다. 새 화면을 푸시하면 이 파일은 자동으로 초기화됩니다.

```jsonl
{"type":"click","choice":"a","text":"Option A - Simple Layout","timestamp":1706000101}
{"type":"click","choice":"c","text":"Option C - Complex Grid","timestamp":1706000108}
{"type":"click","choice":"b","text":"Option B - Hybrid","timestamp":1706000115}
```

전체 이벤트 스트림은 사용자의 탐색 경로를 보여줍니다. 사용자는 최종 결정을 내리기 전에 여러 옵션을 클릭해 볼 수 있습니다. 대개 마지막 `choice` 이벤트가 최종 선택이지만, 클릭 패턴을 보면 추가로 질문해 볼 만한 망설임이나 선호도를 발견할 수 있습니다.

`$STATE_DIR/events` 파일이 존재하지 않는다면 사용자가 브라우저와 상호작용하지 않은 것이므로 터미널 텍스트 응답만 사용하세요.

## 디자인 팁 (Design Tips)

- **질문에 맞춰 충실도(fidelity)를 조절하세요** — 레이아웃 질문에는 와이어프레임을, 디자인 다듬기 질문에는 정교한 스타일링을 사용하세요.
- **각 페이지에 질문을 명확히 설명하세요** — 단순히 "하나를 고르세요"가 아니라 "어느 레이아웃이 더 전문적으로 느껴지나요?"처럼 구체적으로 적으세요.
- **다음으로 넘어가기 전에 충분히 개선(Iterate)하세요** — 피드백으로 인해 현재 화면이 바뀌어야 한다면 새 버전을 작성하세요.
- 화면당 **최대 2~4개의 옵션**만 제시하세요.
- **중요할 때는 실제 콘텐츠를 사용하세요** — 사진 포트폴리오 사이트라면 실제 이미지(Unsplash 등)를 사용하세요. 플레이스홀더 콘텐츠는 디자인상의 문제를 가립니다.
- **목업은 단순하게 유지하세요** — 픽셀 단위의 완벽한 디자인보다는 레이아웃과 구조에 집중하세요.

## 파일 명명 규칙 (File Naming)

- 의미 있는(semantic) 이름을 사용하세요: `platform.html`, `visual-style.html`, `layout.html`
- 절대 파일 이름을 재사용하지 마세요 — 각 화면은 반드시 새 파일이어야 합니다.
- 수정 반복(iterations) 시에는 버전 접미사를 붙이세요: `layout-v2.html`, `layout-v3.html`
- 서버는 수정 시간(modification time) 기준으로 가장 최신 파일을 서빙합니다.

## 정리하기 (Cleaning Up)

```bash
bash scripts/stop-server.sh $SESSION_DIR
```

세션에서 `--project-dir`을 사용했다면 목업 파일들은 추후 참조할 수 있도록 `.superpowers/brainstorm/`에 그대로 보존됩니다. `/tmp`에 생성된 세션만 종료 시 삭제됩니다.

## 레퍼런스 (Reference)

- 프레임 템플릿 (CSS 레퍼런스): `scripts/frame-template.html`
- 헬퍼 스크립트 (클라이언트 사이드): `scripts/helper.js`
