# GitHub 이슈 (GitHub issues)

`gh`가 설치되어 있고 인증된 상태라면 `gh`를 사용하세요. 인증, 속도 제한(rate limits), JSON 처리를 자동으로 해결해 줍니다. `gh`를 쓸 수 없다면 `curl`을 이용한 공개 API로 폴백(fall back)하고, 그마저도 안 되면 인간 파트너(human partner)가 직접 열 수 있는 URL을 전달하세요.

## 검색 (Search)

```bash
gh search issues --repo obra/superpowers --limit 10 "<terms>" \
  --json number,state,title --jq '.[] | "\(.number)\t\(.state)\t\(.title)"'
```

`gh`가 없는 경우 (비인증 상태, 분당 10회 요청 제한):

```bash
curl -s -H "Accept: application/vnd.github+json" \
  "https://api.github.com/search/issues?q=repo:obra/superpowers+is:issue+<url-encoded terms>&per_page=10" \
  | jq -r '.items[] | "\(.number)\t\(.state)\t\(.title)"'
```

`curl`도 없다면 `https://github.com/obra/superpowers/issues?q=<terms>` 링크를 전달하세요.

## 이슈 등록 (File)

내용을 모두 채운 `templates/issue.md`를 작업 공간(workspace)에 기록하고 정확한 전문을 파트너에게 보여주세요. 승인을 받은 후 다음을 실행하세요:

```bash
gh issue create --repo obra/superpowers --title "<title>" --body-file <path> \
  --label bug --label automated-issue-report
```

제출자에게 푸시(push) 권한이 없으면 GitHub는 라벨을 조용히 무시하므로, 라벨은 저장소 협력자(collaborator)인 경우에만 실제로 적용됩니다. 하지만 템플릿 하단 푸터(footer)를 통해 이 이슈가 스킬을 통해 등록되었음은 여전히 표시됩니다. `gh`로는 파일을 첨부할 수 없으므로, 이슈가 생성된 후 브라우저를 통해 직접 첨부할 수 있도록 파트너에게 번들 경로를 알려주세요.

`gh`가 없는 경우, 누구에게나 `bug` 라벨을 자동으로 적용해 주는 `bug_report.md` 템플릿 기반의 미리 채워진(prefilled) 링크를 전달하세요:

```
https://github.com/obra/superpowers/issues/new?template=bug_report.md&title=<url-encoded title>&body=<url-encoded body>
```

GitHub는 약 8,000자를 초과하는 URL을 거부합니다. 길이가 이를 넘어가면 제목(title)만 포함된 링크를 전달하고 본문(body)은 파일에서 복사해 붙여넣도록 파트너에게 안내하세요.
