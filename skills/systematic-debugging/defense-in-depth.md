# 심층 방어 유효성 검사 (Defense-in-Depth Validation)

## 개요

잘못된 데이터로 인해 발생한 버그를 수정할 때, 한 곳에만 유효성 검사를 추가하면 충분한 것처럼 느껴집니다. 하지만 그 단일 검사는 다른 코드 경로, 리팩터링, 또는 모의 객체(mock)에 의해 우회될 수 있습니다.

**핵심 원칙:** 데이터가 통과하는 **모든(EVERY)** 계층에서 유효성을 검사하세요. 버그가 발생하는 것을 구조적으로 불가능하게 만드세요.

## 왜 다중 계층인가

단일 유효성 검사: "버그를 고쳤다"
다중 계층 검사: "버그가 불가능하게 만들었다"

서로 다른 계층은 서로 다른 케이스를 잡아냅니다:
- 진입점(Entry) 유효성 검사는 대부분의 버그를 잡아냅니다
- 비즈니스 로직은 엣지 케이스를 잡아냅니다
- 환경 가드(Environment guards)는 특정 컨텍스트에서의 위험을 방지합니다
- 디버그 로깅은 다른 계층이 실패했을 때 원인 파악을 돕습니다

## 4가지 계층 (The Four Layers)

### 계층 1: 진입점 유효성 검사 (Layer 1: Entry Point Validation)
**목적:** API 경계에서 명백히 잘못된 입력을 거부합니다

```typescript
function createProject(name: string, workingDirectory: string) {
  if (!workingDirectory || workingDirectory.trim() === '') {
    throw new Error('workingDirectory cannot be empty');
  }
  if (!existsSync(workingDirectory)) {
    throw new Error(`workingDirectory does not exist: ${workingDirectory}`);
  }
  if (!statSync(workingDirectory).isDirectory()) {
    throw new Error(`workingDirectory is not a directory: ${workingDirectory}`);
  }
  // ... 진행
}
```

### 계층 2: 비즈니스 로직 유효성 검사 (Layer 2: Business Logic Validation)
**목적:** 데이터가 해당 작업의 맥락에 맞는지 확인합니다

```typescript
function initializeWorkspace(projectDir: string, sessionId: string) {
  if (!projectDir) {
    throw new Error('projectDir required for workspace initialization');
  }
  // ... 진행
}
```

### 계층 3: 환경 가드 (Layer 3: Environment Guards)
**목적:** 특정 컨텍스트에서 위험한 작업이 수행되는 것을 방지합니다

```typescript
async function gitInit(directory: string) {
  // 테스트 환경에서는 임시 디렉터리 외부에서의 git init을 거부함
  if (process.env.NODE_ENV === 'test') {
    const normalized = normalize(resolve(directory));
    const tmpDir = normalize(resolve(tmpdir()));

    if (!normalized.startsWith(tmpDir)) {
      throw new Error(
        `Refusing git init outside temp dir during tests: ${directory}`
      );
    }
  }
  // ... 진행
}
```

### 계층 4: 디버그 계측 (Layer 4: Debug Instrumentation)
**목적:** 사후 분석(forensics)을 위해 컨텍스트를 캡처합니다

```typescript
async function gitInit(directory: string) {
  const stack = new Error().stack;
  logger.debug('About to git init', {
    directory,
    cwd: process.cwd(),
    stack,
  });
  // ... 진행
}
```

## 패턴 적용하기

버그를 발견했을 때:

1. **데이터 흐름 추적** - 잘못된 값이 어디서 시작되는가? 어디서 사용되는가?
2. **모든 체크포인트 매핑** - 데이터가 통과하는 모든 지점을 나열합니다
3. **각 계층에 유효성 검사 추가** - 진입점, 비즈니스, 환경, 디버그 계층
4. **각 계층 테스트** - 계층 1을 우회해 보고 계층 2가 잡아내는지 확인합니다

## 실제 세션 예시

버그: 빈 `projectDir`로 인해 소스 코드 디렉터리에서 `git init`이 실행됨

**데이터 흐름:**
1. 테스트 설정(setup) → 빈 문자열
2. `Project.create(name, '')`
3. `WorkspaceManager.createWorkspace('')`
4. `git init`이 `process.cwd()`에서 실행됨

**추가된 4가지 계층:**
- 계층 1: `Project.create()`에서 비어 있지 않은지/존재하는지/쓰기 가능한지 검증
- 계층 2: `WorkspaceManager`에서 `projectDir`가 비어 있지 않은지 검증
- 계층 3: `WorktreeManager`에서 테스트 중 `tmpdir` 외부의 `git init` 거부
- 계층 4: `git init` 실행 전 스택 트레이스 로깅

**결과:** 1,847개의 모든 테스트가 통과했으며, 버그 재현이 불가능해짐

## 핵심 통찰 (Key Insight)

네 가지 계층이 모두 필요했습니다. 테스트하는 동안 각 계층은 다른 계층이 놓친 버그들을 잡아냈습니다:
- 다른 코드 경로들이 진입점 유효성 검사를 우회함
- 모의 객체(mock)들이 비즈니스 로직 검사를 우회함
- 다른 플랫폼의 엣지 케이스들은 환경 가드가 필요했음
- 디버그 로깅이 구조적인 오용(structural misuse)을 찾아냄

**단 하나의 유효성 검사 지점에서 멈추지 마세요.** 모든 계층에 검사를 추가하세요.
