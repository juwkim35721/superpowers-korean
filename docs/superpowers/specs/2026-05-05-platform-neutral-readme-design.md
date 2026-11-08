# 플랫폼 중립적 README 정렬 — Phase C 설계 (Platform-neutral README ordering — Phase C design)

## 배경

Phase A와 Phase B(`2026-05-05-platform-neutral-prose-design.md` 및 `2026-05-05-platform-neutral-config-refs-design.md` 참조)를 통해 이미 README 내의 일반적인 Claude 문구와 설정 파일 참조를 중립화했습니다. 특정 플랫폼에 치우친 마지막 신호는 레이아웃입니다: README의 두 플랫폼 목록은 Claude Code를 맨 앞에 두고 있으며 다른 항목들도 엄격한 알파벳순으로 정렬되어 있지 않습니다.

이번 단계에서는 정렬 순서를 수정합니다. 본문 문구 변경은 없습니다.

## 범위 내 (In scope)

1. **빠른 시작(Quickstart) 플랫폼 목록** (`README.md:7`) — 지원되는 하네스들의 인라인 링크 목록
2. **설치(Installation) 섹션 순서** (`README.md:35–152`) — 하네스별 설치 하위 섹션들

## 범위 외 (Out of scope)

- 본문 문구, 마켓플레이스 이름, 플러그인 ID, URL — 모두 현재 상태 그대로 사실에 부합합니다.
- Claude Code 섹션의 시각적 비중 (공식 Anthropic 마켓플레이스와 Superpowers 마켓플레이스라는 두 개의 하위 섹션을 가짐). 둘 다 실제 설치 경로이므로, 이를 하나로 합치면 정확한 정보를 숨기게 됩니다.
- 각 설치 블록 내부의 섹션 제목 및 콘텐츠 — 블록들의 배치 순서만 변경됩니다.

## 치환 (Substitution)

두 목록 모두 엄격한 알파벳순으로 재정렬합니다:

| 기존 순서 | 새로운 순서 |
|---|---|
| Claude Code | Claude Code |
| Codex CLI | Codex App |
| Codex App | Codex CLI |
| Factory Droid | Cursor |
| Gemini CLI | Factory Droid |
| OpenCode | Gemini CLI |
| Cursor | GitHub Copilot CLI |
| GitHub Copilot CLI | OpenCode |

세 가지 이동이 발생합니다: Codex App과 Codex CLI의 자리가 바뀌고, Cursor가 두 칸 위로 올라가며, GitHub Copilot CLI가 한 칸 위로 올라갑니다.

Claude Code는 알파벳 순서상 우연히 첫 번째 자리를 유지합니다 (`Cl…`이 `Co…`보다 앞서므로).

## 커밋 계획

빠른 시작 목록과 설치 섹션 중 하나만 변경하면 두 목록 사이에 불일치가 발생하므로, 두 목록을 모두 포함하는 하나의 원자적(atomic) 커밋으로 진행합니다.

## 검증 (Verification)

- 빠른 시작 앵커(`#claude-code`, `#codex-app` 등)가 여전히 기존 `### …` 제목들로 정상 연결되는지 확인합니다 — 변경된 제목은 없습니다.
- 각 설치 하위 섹션의 본문은 변경 전후가 바이트 단위로 동일해야 하며, 오직 위치만 변경됩니다.
- `git diff README.md`가 콘텐츠 편집 없이 섹션 이동만 보여주는지 확인합니다.
