# Claude Code를 위한 크로스 플랫폼 폴리글랏 훅 (Polyglot Hooks)

Claude Code 플러그인에는 Windows, macOS, Linux에서 모두 작동하는 훅(hooks)이 필요합니다. 이 문서는 `hooks/run-hook.cmd`에 사용된 단일 범용 디스패처(dispatcher) 패턴을 설명합니다.

> **공식 기준 소스(Authoritative source):** `hooks/run-hook.cmd`가 정식 구현체입니다. 이 문서와 코드가 다를 경우 코드를 신뢰하세요.

## 문제점 (The Problem)

Claude Code는 셸을 통해 훅 명령어를 실행합니다:
- **macOS/Linux**: bash 또는 sh
- **Git Bash가 설치된 Windows**: Git Bash
- **Git Bash가 없는 Windows**: PowerShell (이전 버전에서는 CMD.exe 사용)

Windows의 두 폴백(fallback) 셸 모두 우리의 명령어 문자열을 파싱하지 못합니다. PowerShell은 따옴표로 묶인 선행 경로를 문자열 표현식으로 처리하여 그다음에 오는 일반 단어(bareword)에서 에러를 내며, CMD.exe의 `/c` 따옴표 규칙은 경로에 `(`와 같은 메타 문자가 포함될 때 바깥쪽 따옴표를 제거해 버립니다. 따라서 우리의 훅은 `"shell": "bash"`(Claude Code 2.1.81부터 지원되며, 이전 버전은 이 키를 무시함)를 선언하여 Git Bash 경로를 강제하고, Git Bash가 없을 때는 셸 파서 실패 대신 "Git for Windows를 설치하세요"라는 실행 가능한 안내 에러를 생성하도록 합니다.

이로 인해 다음과 같은 여러 과제가 발생합니다:

1. **스크립트 실행**: Windows CMD는 `.sh` 파일을 직접 실행할 수 없습니다
2. **경로 형식**: Windows는 백슬래시(`C:\path`)를 사용하고, Unix는 슬래시(`/path`)를 사용합니다
3. **환경 변수**: CMD에서는 `$VAR` 문법이 작동하지 않습니다
4. **`.sh` 자동 접두사 추가(auto-prepend)**: Windows용 Claude Code는 경로에 `.sh`가 포함된 모든 명령어 앞에 자동으로 `bash`를 붙입니다 — 스크립트에 확장자가 있으면 디스패처와 충돌을 일으킵니다

## 해결책: 확장자 없는 스크립트 + 단일 범용 디스패처

이 저장소는 모든 훅에 대해 하나의 범용 `run-hook.cmd` 디스패처를 사용합니다. 훅 스크립트는 **확장자가 없습니다**(`session-start.sh`가 아니라 `session-start`). 이는 의도적인 설계로, Claude Code의 Windows 자동 감지 기능이 디스패처 명령어 앞에 `bash`를 붙여 명령어를 망가뜨리는 것을 방지합니다.

### 파일 구조 (File Structure)

```
hooks/
├── hooks.json          # 확장자 없는 스크립트 이름과 함께 run-hook.cmd를 가리킴
├── run-hook.cmd        # 크로스 플랫폼 디스패처 (폴리글랏 래퍼)
└── session-start       # 실제 훅 로직 — 확장자 없는 bash 스크립트
```

### hooks.json

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|clear|compact",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd\" session-start",
            "powershell": "& \"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd\" session-start",
            "shell": "bash",
            "async": false
          }
        ]
      }
    ]
  }
}
```

`${CLAUDE_PLUGIN_ROOT}`에 공백이 포함될 수 있으므로 경로를 따옴표로 감쌉니다.

### `powershell` 키

Windows 환경의 GitHub Copilot(CLI 및 VS Code)은 `shell` 키와 상관없이 PowerShell을 통해 훅 명령어를 실행하며(#2189), 이때 따옴표로 묶인 선행 경로가 문자열 표현식으로 파싱되어 뒤따르는 일반 단어에서 에러("Unexpected token 'session-start'")가 발생합니다. Copilot은 `powershell` 키가 존재하면 `command` 대신 이를 사용하며, `&` 호출 연산자를 통해 동일한 디스패치를 수행합니다. Claude Code는 이 키를 무시합니다.

## `run-hook.cmd`의 상위 수준 작동 방식

`run-hook.cmd`는 폴리글랏(polyglot) 스크립트입니다. Windows는 첫 번째 블록을 배치(batch) 명령어로 처리하는 반면, Unix 셸은 해당 블록을 아무 동작도 하지 않는(no-op) 히어독(heredoc)으로 처리하고 그 이후부터 실행을 이어갑니다.

이 문서에서 구현 코드를 복사하지 마세요. 디스패처를 수정할 때는 `hooks/run-hook.cmd`를 직접 읽고, 수정 후에는 `tests/hooks/test-session-start.sh`를 실행하세요.

### Windows(CMD.exe)에서의 작동 방식

1. 배치 섹션은 스크립트 이름을 검증하고 디스패처 자신의 위치로부터 훅 디렉토리 경로를 확인합니다.
2. 다음 네 곳에서 순서대로 bash를 찾습니다:
   - `C:\Program Files\Git\bin\bash.exe`
   - `C:\Program Files (x86)\Git\bin\bash.exe`
   - `%LOCALAPPDATA%\Programs\Git\bin\bash.exe` (사용자별 Git for Windows 설치 경로; `LOCALAPPDATA`가 설정된 경우에만 확인)
   - `PATH`에 있는 `bash` (MSYS2, Cygwin 또는 기본 경로가 아닌 Git 설치). 현재 디렉토리에서 `where`나 `bash`가 실행되지 않도록 `%SystemRoot%\System32\where.exe $PATH:bash`를 실행하여 찾습니다. WSL 런처(`System32`, `Sysnative` 또는 `%LOCALAPPDATA%\Microsoft\WindowsApps`에 있는 `bash.exe`)와 확장자가 없는 일치 항목은 Linux 배포판이 설치되어 있지 않을 때 실패하므로 건너뜁니다.
3. bash를 찾으면 훅 디렉토리에서 지정된 확장자 없는 훅 스크립트를 실행하고 해당 스크립트의 종료 코드로 종료합니다.
4. bash를 찾지 못하면 디스패처는 조용히 `0`으로 종료합니다 — 플러그인은 훅만 건너뛴 채 계속 정상 작동합니다.
5. `exit /b`가 Unix 섹션에 도달하기 전에 CMD를 중단시킵니다.

### Unix(bash/sh)에서의 작동 방식

1. `: << 'CMDBLOCK'`이 아무 동작도 하지 않는(no-op) 명령어에 대해 히어독을 엽니다.
2. 전체 CMD 배치 블록이 히어독에 의해 소비되고 무시됩니다.
3. `CMDBLOCK` 이후에 bash가 스크립트 디렉토리를 확인하고 지정된 확장자 없는 스크립트를 직접 `exec`합니다.

### 주요 설계 결정 (Key design decisions)

| 결정 사항 | 이유 |
|----------|-----|
| 확장자 없는 스크립트 | Claude Code의 Windows `.sh` 자동 접두사 추가 기능이 디스패처 명령어와 충돌하는 것을 방지함 |
| `-l` (로그인 셸) 미사용 | 불필요함; 훅 스크립트는 자체 완결적이어야 하며 로그인 셸의 PATH 설정에 의존해서는 안 됨 |
| `cygpath` 미사용 | Bash는 Windows 경로를 직접 받아도 올바르게 처리함; `cygpath`는 과거의 `-c "..."` 호출 패턴에서 필요했던 것이며 직접 exec할 때는 필요하지 않음 |
| bash 미발견 시 조용한 종료 | Git for Windows가 없는 사용자에게 플러그인이 깨지는 것을 방지함; 훅 컨텍스트 주입만 우아하게 건너뜀 |

## 크로스 플랫폼 훅 스크립트 작성하기

훅 로직은 확장자 없는 스크립트 파일에 작성합니다. 이식성을 높이는 몇 가지 패턴은 다음과 같습니다:

### 권장 사항 (Do)
- 가능한 한 순수 bash 내장 기능(builtins)을 사용하세요
- 백틱(backticks) 대신 `$(command)`를 사용하세요
- 모든 변수 확장을 따옴표로 감싸세요: `"$VAR"`

### 피해야 할 사항 (Avoid)
- 폴백 없이 PATH에 의존하는 도구에 기대지 마세요 (훅은 `-l` 없이 실행되므로 로그인 셸 PATH가 설정되지 않습니다)
- 스크립트에 `.sh` 확장자를 붙이지 마세요 — Claude Code의 Windows 자동 접두사 추가 기능을 유발합니다

### 예시: 외부 도구 없이 JSON 이스케이프 처리하기

```bash
escape_for_json() {
    local input="$1"
    local output=""
    local i char
    for (( i=0; i<${#input}; i++ )); do
        char="${input:$i:1}"
        case "$char" in
            $'\\') output+='\\' ;;
            '"') output+='\"' ;;
            $'\n') output+='\n' ;;
            $'\r') output+='\r' ;;
            $'\t') output+='\t' ;;
            *) output+="$char" ;;
        esac
    done
    printf '%s' "$output"
}
```

## 문제 해결 (Troubleshooting)

### "bash is not recognized"

CMD가 디스패처가 시도하는 네 곳의 위치 어디에서도 bash를 찾지 못한 것입니다. 디스패처는 에러를 내는 대신 조용히 종료(0)하므로 훅이 건너뛰어집니다. Git for Windows를(시스템 전체 또는 사용자별로) 설치하거나 WSL이 아닌 `bash`가 `PATH`에 있는지 확인하세요.

### 훅이 Unix에서는 실행되지만 Windows에서는 아무 동작도 하지 않는 경우

`hooks.json`에서 스크립트 파일 이름에 **확장자가 없는지** 확인하세요. `run-hook.cmd session-start.sh`와 같은 명령어는 Claude Code의 `.sh` 자동 감지를 유발하여 의도된 CMD 디스패처 경로를 우회하거나, 존재하지 않는 `session-start.sh` 스크립트 실행을 시도할 수 있습니다.

### 훅이 아예 실행되지 않는 경우

`hooks.json`의 `matcher`가 하네스가 방출하는 이벤트 타입과 일치하는지 확인하세요. Claude Code는 `startup|clear|compact`를 사용하고, Cursor는 `sessionStart`를 사용합니다. Cursor 변형 설정은 `hooks-cursor.json`을 확인하세요.

## 관련 이슈 (Related Issues)

- [anthropics/claude-code#9758](https://github.com/anthropics/claude-code/issues/9758) — Windows에서 `.sh` 스크립트가 에디터에서 열리는 문제
- [anthropics/claude-code#3417](https://github.com/anthropics/claude-code/issues/3417) — Windows에서 훅이 작동하지 않는 문제
