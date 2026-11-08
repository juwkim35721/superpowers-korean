# Hermes 버전 범프 연결 설계 (Hermes Version-Bump Wiring Design)

**날짜:** 2026-08-05
**수정일:** 2026-08-06
**상태:** 승인됨 (Approved)

## 목표

`.version-bump.json`에 `.hermes-plugin/plugin.yaml`을 등록하고, Bash로 직접 YAML 파서를 구현하지 않고도 `scripts/bump-version.sh`가 YAML을 처리할 수 있도록 가르침으로써 `.hermes-plugin/plugin.yaml`을 저장소 버전과 완벽하게 동기화된 상태(lockstep)로 유지합니다.

## 설계

- `.version-bump.json`에 `{ "path": ".hermes-plugin/plugin.yaml", "field": "version" }`을 추가합니다.
- `.json`은 기존의 `jq` 헬퍼를 통해 라우팅하고, `.yaml`은 Mike Farah의 `yq` v4를 통해 라우팅합니다. YAML 키와 값은 표현식에 직접 문자열 보간(interpolate)하지 않고 데이터로 전달합니다.
- 존재하는 최상위(top-level) YAML 문자열 필드만 지원합니다. 중첩된 필드와 `.yml` 확장자는 범위 외입니다.
- `--check`, `--audit` 및 버전 업데이트를 모두 동일한 소형 읽기/쓰기 디스패처를 통해 라우팅합니다.
- 버전 범프가 매니페스트 파일에 쓰기 작업을 수행하기 전에, 필수 도구를 검증하고 선언된 모든 존재하는 매니페스트를 디스패처를 통해 읽어보는 읽기 전용 사전 검사(preflight)를 1회 실행합니다. 이를 통해 앞선 JSON 파일들이 이미 업데이트된 후에 결정론적인 YAML 실패가 발생하는 상황을 방지합니다. 누락된 파일에 대한 동작은 변경되지 않으며, `--help`는 `jq`나 `yq` 없이도 여전히 작동합니다.

사전 검사(preflight)는 신뢰성을 위해 추가된 유일한 기능입니다. 이는 스크립트를 트랜잭션 방식으로 만들거나 기존의 감사(audit) 및 에러 상태 동작을 재설계하지 않습니다.

## 테스트

세 가지 집중된 행동 테스트가 격리된 임시 픽스처에 대해 실제 스크립트를 실행하여 다음을 증명합니다:

- 정렬된(버전이 일치하는) JSON과 YAML이 `--check`와 `--audit`를 통과하고, 버전 범프가 두 포맷을 모두 업데이트함;
- JSON이 먼저 선언되고 최상위 `version`이 문자열이 아닌 YAML 매니페스트가 뒤에 오는 상태에서 실제 범프를 실행하면 0이 아닌 종료 코드(nonzero)로 종료되며 모든 매니페스트 파일이 바이트 단위로 변경되지 않은 채 유지됨; 그리고
- 실제 `.version-bump.json`이 Hermes 매니페스트를 등록하고 있음.

또한 검증 과정에서 저장소에 대해 셸 린트(shell lint)와 `scripts/bump-version.sh --check`를 실행합니다.

## 비목표 (Non-Goals)

- 직접 작성한(hand-written) YAML 파서를 사용하지 않습니다.
- `.yml` 또는 중첩된 YAML을 지원하지 않습니다.
- Hermes 런타임을 변경하지 않습니다.
- 롤백 프레임워크, 범용 설정 스키마 계층, 감사/상태 리팩토링 또는 전체 실패 매트릭스를 구현하지 않습니다.
- 리뷰 중에 발견된 별개의 버전 검증 및 JSON 표현식 이슈는 여기서 변경하지 않습니다.
