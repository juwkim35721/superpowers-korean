먼저 `prompts/analyst-common.md`를 읽으세요. 해당 파일에 당신의 역할, 입력값, 컨텍스트 안전 규칙, 반환 형식이 정의되어 있습니다. 이 파일은 분석 차원(dimension)을 추가합니다.

분석 차원(Dimension): 차질 및 실패 (Stumbles)

세션이 앞으로 나아가지 못하고 주춤하거나 막혔던 모든 지점을 찾으세요.

각 소스에 대해 케이스 파일의 증거가 확인된 레코드 의미와 추출 명령어를 사용하여 줄 번호를 찾으세요:
- 에러로 표시된 도구 결과, 0이 아닌 종료 코드(non-zero exits), 또는 명시적인 실패 레코드;
- 실패한 셸 명령어(결과에 0이 아닌 종료 코드가 있거나, "command not found", "No such file" 등이 포함된 경우);
- 재시도(retries): 에러 발생 후 동일한 턴 내에서 다시 실행된 동일한 도구 호출;
- 되돌려진 편집(reverted edits): 편집 직후 이전 내용을 복원하는 편집이 이어지거나, 세션이 건드린 파일에 대해 `git checkout`/`git restore`/`git revert`/`git reset`을 실행한 경우;
- 어시스턴트 텍스트 내의 번복/역추적(backtracking: "actually", "let me instead", "that was wrong", "I misread" 등);
- 인간의 교정(human corrections): 어시스턴트의 직전 행동을 반박하거나 바로잡는 인간 프롬프트;
- 권한 거부(permission denials), 훅 실패, API 에러, 속도 제한(rate limits), 중단된 턴(aborted turns), 그리고 작업 도중 발생한 컨텍스트 오버플로우나 컴팩션.

각 차질(stumble)에 대해 줄 번호, 턴, 무엇이 실패했는지, 그리고 그 다음에 무슨 일이 일어났는지(동일한 턴에서 복구됨 / 이후 N번째 줄에서 복구됨 / 끝내 복구되지 않음)를 보고하세요. 동일하게 반복된 실패들은 하나의 분석 결과(finding)로 묶고 횟수를 표시하세요.
