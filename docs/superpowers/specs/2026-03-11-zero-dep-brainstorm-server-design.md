# 제로 의존성 브레인스토밍 서버 (Zero-Dependency Brainstorm Server)

브레인스토밍 컴패니언 서버에 벤더링된 node_modules(express, ws, chokidar — 추적 파일 714개)를 Node.js 내장 모듈만 사용하는 단일 제로 의존성 `server.js`로 대체합니다.

## 동기 (Motivation)

git 저장소에 node_modules를 벤더링하는 것은 공급망(supply chain) 위험을 초래합니다: 고정된 의존성은 보안 패치를 받지 못하고, 714개의 서드파티 코드 파일이 감사 없이 커밋되며, 벤더링된 코드에 대한 수정 사항이 일반 커밋처럼 보이게 됩니다. 실제 위험도는 낮지만(localhost 전용 개발 서버이므로), 이를 제거하는 것은 매우 간단합니다.

## 아키텍처

`http`, `crypto`, `fs`, `path`만 사용하는 단일 `server.js` 파일(약 250~300줄)로 구성됩니다. 이 파일은 두 가지 역할을 수행합니다:

- **직접 실행될 때** (`node server.js`): HTTP/WebSocket 서버를 시작합니다.
- **모듈로 불러올 때** (`require('./server.js')`): 단위 테스트를 위해 WebSocket 프로토콜 함수들을 내보냅니다(export).

### WebSocket 프로토콜

텍스트 프레임에 대해서만 RFC 6455를 구현합니다:

**핸드셰이크 (Handshake):** SHA-1과 RFC 6455 매직 GUID를 사용하여 클라이언트의 `Sec-WebSocket-Key`로부터 `Sec-WebSocket-Accept`를 계산합니다. 101 Switching Protocols를 반환합니다.

**프레임 디코딩 (클라이언트 -> 서버):** 마스킹된 세 가지 길이 인코딩을 처리합니다:
- 소형 (Small): 페이로드 < 126바이트
- 중형 (Medium): 126~65535바이트 (16비트 확장)
- 대형 (Large): > 65535바이트 (64비트 확장)

4바이트 마스크 키를 사용하여 페이로드를 XOR 언마스킹합니다. `{ opcode, payload, bytesConsumed }`를 반환하거나 불완전한 버퍼의 경우 `null`을 반환합니다. 마스킹되지 않은 프레임은 거부합니다.

**프레임 인코딩 (서버 -> 클라이언트):** 동일한 세 가지 길이 인코딩을 사용하는 언마스킹된 프레임입니다.

**처리하는 Opcode:** TEXT (0x01), CLOSE (0x08), PING (0x09), PONG (0x0A). 인식할 수 없는 opcode는 상태 코드 1003(Unsupported Data)과 함께 close 프레임을 받습니다.

**의도적으로 생략한 기능:** 바이너리 프레임, 단편화된(fragmented) 메시지, 확장(permessage-deflate), 서브프로토콜. 이러한 기능들은 localhost 클라이언트 간의 작은 JSON 텍스트 메시지에는 불필요합니다. 확장과 서브프로토콜은 핸드셰이크 과정에서 협상되므로, 이를 광고(advertise)하지 않으면 절대 활성화되지 않습니다.

**버퍼 누적 (Buffer accumulation):** 각 연결은 자체 버퍼를 유지합니다. `data` 이벤트 발생 시 버퍼에 데이터를 추가하고, `decodeFrame`이 null을 반환하거나 버퍼가 빌 때까지 루프를 돕니다.

### HTTP 서버

세 가지 라우트를 제공합니다:

1. **`GET /`** — 화면 디렉터리에서 수정 시간(mtime) 기준 가장 최신 `.html` 파일을 제공합니다. 전체 문서(full document)와 프래그먼트(fragment)를 구분하고, 프래그먼트는 프레임 템플릿으로 감싼 뒤 `helper.js`를 주입합니다. `text/html`을 반환합니다. `.html` 파일이 없을 때는 `helper.js`가 주입된 하드코딩된 대기 페이지("Waiting for Claude to push a screen...")를 제공합니다.
2. **`GET /files/*`** — 하드코딩된 확장자 맵(html, css, js, png, jpg, gif, svg, json)에서 MIME 타입을 조회하여 화면 디렉터리의 정적 파일을 제공합니다. 파일이 없으면 404를 반환합니다.
3. **그 외 모든 요청** — 404.

WebSocket 업그레이드는 요청 핸들러와 분리되어 HTTP 서버의 `'upgrade'` 이벤트를 통해 처리됩니다.

### 설정 (Configuration)

환경 변수 (모두 선택 사항):

- `BRAINSTORM_PORT` — 바인딩할 포트 (기본값: 49152~65535 사이의 무작위 높은 포트)
- `BRAINSTORM_HOST` — 바인딩할 인터페이스 (기본값: `127.0.0.1`)
- `BRAINSTORM_URL_HOST` — 시작 JSON의 URL에 사용할 호스트 이름 (기본값: 호스트가 `127.0.0.1`일 때는 `localhost`, 그 외에는 호스트와 동일)
- `BRAINSTORM_DIR` — 화면 디렉터리 경로 (기본값: `/tmp/brainstorm`)

### 시작 시퀀스 (Startup Sequence)

1. `SCREEN_DIR`가 존재하지 않으면 생성합니다 (`mkdirSync` recursive).
2. `__dirname`에서 프레임 템플릿과 `helper.js`를 로드합니다.
3. 설정된 호스트/포트에서 HTTP 서버를 시작합니다.
4. `SCREEN_DIR`에 대해 `fs.watch`를 시작합니다.
5. 리스닝에 성공하면 stdout에 `server-started` JSON을 로깅합니다: `{ type, port, host, url_host, url, screen_dir }`
6. 백그라운드 실행 등으로 stdout이 숨겨진 경우에도 에이전트가 연결 정보를 찾을 수 있도록 동일한 JSON을 `SCREEN_DIR/.server-info`에 기록합니다.

### 애플리케이션 레벨 WebSocket 메시지

클라이언트로부터 TEXT 프레임이 도착했을 때:

1. JSON으로 파싱합니다. 파싱에 실패하면 stderr에 로깅하고 계속 진행합니다.
2. stdout에 `{ source: 'user-event', ...event }` 형태로 로깅합니다.
3. 이벤트에 `choice` 속성이 포함되어 있으면 해당 JSON을 `SCREEN_DIR/.events`에 추가합니다 (이벤트당 한 줄).

### 파일 감시 (File Watching)

`fs.watch(SCREEN_DIR)`가 chokidar를 대체합니다. HTML 파일 이벤트 발생 시:

- 새 파일 생성 시 (존재하는 파일에 대한 `rename` 이벤트): `.events` 파일이 있으면 삭제(`unlinkSync`)하고, stdout에 `screen-added`를 JSON으로 로깅합니다.
- 파일 변경 시 (`change` 이벤트): stdout에 `screen-updated`를 JSON으로 로깅합니다 (`.events`는 초기화하지 **않음**).
- 두 이벤트 모두: 연결된 모든 WebSocket 클라이언트에 `{ type: 'reload' }`를 전송합니다.

macOS와 Linux에서 흔히 발생하는 중복 이벤트를 방지하기 위해 파일 이름별로 약 100ms 타임아웃의 디바운스(debounce)를 적용합니다.

### 에러 처리

- WebSocket 클라이언트의 잘못된 형식의 JSON: stderr에 로깅하고 계속 진행
- 처리되지 않은 opcode: 상태 코드 1003으로 연결 종료
- 클라이언트 연결 해제: 브로드캐스트 세트에서 제거
- `fs.watch` 에러: stderr에 로깅하고 계속 진행
- 우아한 종료(graceful shutdown) 로직 없음 — 셸 스크립트가 SIGTERM을 통해 프로세스 수명주기를 처리함

## 변경 사항 요약

| 변경 전 | 변경 후 |
|---|---|
| `index.js` + `package.json` + `package-lock.json` + 714개의 `node_modules` 파일 | `server.js` (단일 파일) |
| express, ws, chokidar 의존성 | 없음 |
| 정적 파일 서빙 없음 | `/files/*`가 화면 디렉터리에서 파일을 서빙함 |

## 유지되는 사항

- `helper.js` — 변경 없음
- `frame-template.html` — 변경 없음
- `start-server.sh` — 한 줄 업데이트: `index.js`를 `server.js`로 변경
- `stop-server.sh` — 변경 없음
- `visual-companion.md` — 변경 없음
- 모든 기존 서버 동작 및 외부 인터페이스 계약(contract)

## 플랫폼 호환성

- `server.js`는 크로스 플랫폼 Node 내장 모듈만 사용합니다.
- `fs.watch`는 macOS, Linux, Windows의 단일 플랫(flat) 디렉터리에서 안정적으로 동작합니다.
- 셸 스크립트는 bash를 필요로 합니다 (Windows의 경우 Claude Code에 필수인 Git Bash 사용).

## 테스트

**단위 테스트** (`ws-protocol.test.js`): `server.js`가 내보낸(export) 함수들을 직접 불러와 WebSocket 프레임 인코딩/디코딩, 핸드셰이크 계산 및 프로토콜 엣지 케이스를 테스트합니다.

**통합 테스트** (`server.test.js`): HTTP 서빙, WebSocket 통신, 파일 감시, 브레인스토밍 워크플로우 등 전체 서버 동작을 테스트합니다. 테스트 전용 클라이언트 의존성으로 `ws` npm 패키지를 사용합니다 (최종 사용자에게는 배포되지 않음).
