# 스킬 작성 모범 사례 (Skill authoring best practices)

> 에이전트가 성공적으로 발견하고 사용할 수 있는 효과적인 스킬(Skill)을 작성하는 방법을 알아봅니다.

좋은 스킬은 간결하고, 구조가 잘 잡혀 있으며, 실제 사용 사례를 통해 테스트된 스킬입니다. 이 가이드는 에이전트가 효과적으로 발견하고 사용할 수 있는 스킬을 작성하는 데 도움이 되는 실용적인 작성 의사결정 기준을 제공합니다.

스킬의 작동 방식에 대한 개념적 배경은 [스킬 개요(Skills overview)](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)를 참조하세요.

## 핵심 원칙

### 간결함이 핵심입니다

[컨텍스트 윈도우(context window)](https://platform.claude.com/docs/en/build-with-claude/context-windows)는 공공재입니다. 작성한 스킬은 에이전트가 알아야 하는 다른 모든 정보와 컨텍스트 윈도우를 공유합니다. 여기에는 다음이 포함됩니다:

* 시스템 프롬프트
* 대화 기록
* 다른 스킬들의 메타데이터
* 사용자의 실제 요청

스킬에 포함된 모든 토큰이 즉각적인 비용을 발생시키는 것은 아닙니다. 시작 시점에는 모든 스킬에서 메타데이터(이름과 설명)만 미리 로드됩니다. 에이전트는 해당 스킬이 관련성을 갖게 될 때만 SKILL.md를 읽고, 추가 파일은 필요할 때만 읽습니다. 하지만 SKILL.md를 간결하게 유지하는 것은 여전히 중요합니다. 에이전트가 일단 파일을 로드하면, 모든 토큰이 대화 기록 및 다른 컨텍스트와 경쟁하기 때문입니다.

**기본 가정**: 에이전트는 이미 매우 똑똑합니다

에이전트가 아직 가지고 있지 않은 컨텍스트만 추가하세요. 각 정보에 대해 다음 질문을 던져보세요:

* "에이전트에게 이 설명이 정말 필요한가?"
* "에이전트가 이 내용을 이미 알고 있다고 가정할 수 있는가?"
* "이 문단은 토큰 비용을 지불할 만큼의 가치가 있는가?"

**좋은 예시: 간결함** (약 50 토큰):

````markdown  theme={null}
## PDF 텍스트 추출

텍스트 추출에는 pdfplumber를 사용하세요:

```python
import pdfplumber

with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```
````

**나쁜 예시: 지나치게 장황함** (약 150 토큰):

```markdown  theme={null}
## PDF 텍스트 추출

PDF(Portable Document Format) 파일은 텍스트, 이미지 및 기타 콘텐츠를 포함하는
일반적인 파일 형식입니다. PDF에서 텍스트를 추출하려면 라이브러리를 사용해야
합니다. PDF 처리를 위한 많은 라이브러리가 있지만, 사용하기 쉽고 대부분의
경우를 잘 처리하는 pdfplumber를 권장합니다. 먼저 pip를 사용하여 설치해야
합니다. 그런 다음 아래 코드를 사용할 수 있습니다...
```

간결한 버전은 에이전트가 PDF가 무엇인지, 그리고 라이브러리가 어떻게 작동하는지 이미 알고 있다고 가정합니다.

### 적절한 자유도 설정하기

작업의 취약성(fragility)과 가변성(variability)에 맞춰 구체성의 수준을 조절하세요.

**높은 자유도** (텍스트 기반 지침):

다음과 같은 경우에 사용하세요:

* 여러 접근 방식이 유효할 때
* 컨텍스트에 따라 의사결정이 달라질 때
* 휴리스틱(경험적 지침)이 접근 방식을 안내할 때

예시:

```markdown  theme={null}
## 코드 리뷰 프로세스

1. 코드 구조와 구성을 분석합니다
2. 잠재적인 버그나 엣지 케이스를 확인합니다
3. 가독성과 유지보수성을 위한 개선 사항을 제안합니다
4. 프로젝트 컨벤션 준수 여부를 검증합니다
```

**중간 자유도** (매개변수가 있는 의사코드 또는 스크립트):

다음과 같은 경우에 사용하세요:

* 선호되는 패턴이 존재할 때
* 어느 정도의 변형이 허용될 때
* 설정(configuration)이 동작에 영향을 미칠 때

예시:

````markdown  theme={null}
## 리포트 생성

이 템플릿을 사용하고 필요에 따라 커스터마이징하세요:

```python
def generate_report(data, format="markdown", include_charts=True):
    # 데이터 처리
    # 지정된 형식으로 출력 생성
    # 선택적으로 시각화 포함
```
````

**낮은 자유도** (구체적인 스크립트, 매개변수가 거의 없거나 전혀 없음):

다음과 같은 경우에 사용하세요:

* 작업이 취약하고 오류가 발생하기 쉬울 때
* 일관성이 매우 중요할 때
* 특정 순서를 반드시 따라야 할 때

예시:

````markdown  theme={null}
## 데이터베이스 마이그레이션

정확히 이 스크립트를 실행하세요:

```bash
python scripts/migrate.py --verify --backup
```

명령어를 수정하거나 추가 플래그를 붙이지 마세요.
````

**비유**: 에이전트를 경로를 탐색하는 로봇이라고 생각해 보세요:

* **양쪽이 절벽인 좁은 다리**: 안전하게 나아갈 수 있는 길은 단 하나뿐입니다. 구체적인 가드레일과 정확한 지침을 제공하세요(낮은 자유도). 예: 정확한 순서로 실행되어야 하는 데이터베이스 마이그레이션.
* **위험 요소가 없는 탁 트인 들판**: 여러 경로가 성공으로 이어집니다. 전반적인 방향만 제시하고 에이전트가 최적의 경로를 찾도록 신뢰하세요(높은 자유도). 예: 컨텍스트에 따라 최선의 접근 방식이 결정되는 코드 리뷰.

### 사용하려는 모든 모델에서 테스트하기

스킬은 모델에 대한 추가 기능으로 작동하므로, 그 효과는 기반 모델에 따라 달라집니다. 함께 사용할 계획인 모든 모델에서 스킬을 테스트하세요.

**모델별 테스트 고려 사항**:

* **Claude Haiku** (빠르고 경제적): 스킬이 충분한 가이드를 제공하는가?
* **Claude Sonnet** (균형 잡힌 성능): 스킬이 명확하고 효율적인가?
* **Claude Opus** (강력한 추론): 스킬이 지나치게 과도한 설명을 피하고 있는가?

Opus에서 완벽하게 작동하는 지침이라도 Haiku에서는 더 많은 세부 정보가 필요할 수 있습니다. 여러 모델에서 스킬을 사용할 계획이라면, 모든 모델에서 잘 작동하는 지침을 목표로 삼으세요.

## 스킬 구조

<Note>
  **YAML 프론트매터(Frontmatter)**: SKILL.md 프론트매터에는 두 가지 필드가 필수입니다:

  * `name` - 사람이 읽을 수 있는 스킬의 이름 (최대 64자)
  * `description` - 스킬이 수행하는 작업과 언제 사용해야 하는지에 대한 한 줄 설명 (최대 1024자)

  전체 스킬 구조에 대한 자세한 내용은 [스킬 개요](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#skill-structure)를 참조하세요.
</Note>

### 명명 규칙 (Naming conventions)

스킬을 더 쉽게 참조하고 논의할 수 있도록 일관된 명명 패턴을 사용하세요. 스킬이 제공하는 활동이나 기능을 명확하게 설명할 수 있도록 스킬 이름에는 **동명사 형태(gerund form, 동사 + -ing)**를 사용하는 것을 권장합니다.

**좋은 명명 예시 (동명사 형태)**:

* "Processing PDFs"
* "Analyzing spreadsheets"
* "Managing databases"
* "Testing code"
* "Writing documentation"

**허용 가능한 대안**:

* 명사구: "PDF Processing", "Spreadsheet Analysis"
* 행동 지향형: "Process PDFs", "Analyze Spreadsheets"

**피해야 할 이름**:

* 모호한 이름: "Helper", "Utils", "Tools"
* 지나치게 일반적인 이름: "Documents", "Data", "Files"
* 스킬 컬렉션 내에서 일관성 없는 패턴

일관된 명명을 사용하면 다음과 같은 작업이 쉬워집니다:

* 문서 및 대화에서 스킬 참조하기
* 스킬이 어떤 역할을 하는지 한눈에 파악하기
* 여러 스킬을 체계적으로 정리하고 검색하기
* 전문적이고 일관성 있는 스킬 라이브러리 유지하기

### 효과적인 설명(description) 작성하기

`description` 필드는 스킬 탐색(discovery)을 가능하게 하며, 스킬이 무엇을 하는지와 언제 사용해야 하는지를 모두 포함해야 합니다.

<Warning>
  **항상 3인칭으로 작성하세요**. 설명(description)은 시스템 프롬프트에 주입되므로, 시점(point-of-view)이 일관되지 않으면 스킬 탐색에 문제가 발생할 수 있습니다.

  * **좋음:** "Processes Excel files and generates reports" (Excel 파일을 처리하고 리포트를 생성합니다)
  * **피해야 함:** "I can help you process Excel files" (제가 Excel 파일 처리를 도와드릴 수 있습니다)
  * **피해야 함:** "You can use this to process Excel files" (이것을 사용하여 Excel 파일을 처리할 수 있습니다)
</Warning>

**구체적으로 작성하고 핵심 용어를 포함하세요**. 스킬이 수행하는 작업과 스킬을 사용해야 하는 구체적인 트리거/컨텍스트를 모두 포함하세요.

각 스킬에는 단 하나의 `description` 필드가 있습니다. 설명은 스킬 선택에 결정적인 역할을 합니다. 에이전트는 잠재적으로 100개 이상일 수 있는 가용 스킬 중에서 올바른 스킬을 선택하기 위해 이 설명을 사용합니다. 설명은 에이전트가 언제 이 스킬을 선택해야 하는지 알 수 있을 만큼 충분한 세부 정보를 제공해야 하며, SKILL.md의 나머지 본문은 구체적인 구현 세부 사항을 제공합니다.

효과적인 예시:

**PDF 처리 스킬:**

```yaml  theme={null}
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
```

**Excel 분석 스킬:**

```yaml  theme={null}
description: Analyze Excel spreadsheets, create pivot tables, generate charts. Use when analyzing Excel files, spreadsheets, tabular data, or .xlsx files.
```

**Git 커밋 도우미 스킬:**

```yaml  theme={null}
description: Generate descriptive commit messages by analyzing git diffs. Use when the user asks for help writing commit messages or reviewing staged changes.
```

다음과 같이 모호한 설명은 피하세요:

```yaml  theme={null}
description: Helps with documents
```

```yaml  theme={null}
description: Processes data
```

```yaml  theme={null}
description: Does stuff with files
```

### 점진적 공개(Progressive disclosure) 패턴

SKILL.md는 온보딩 가이드의 목차처럼 필요에 따라 에이전트를 상세 자료로 안내하는 개요 역할을 합니다. 점진적 공개가 작동하는 방식에 대한 설명은 개요 문서의 [스킬 작동 방식(How Skills work)](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#how-skills-work)을 참조하세요.

**실용적인 가이드:**

* 최적의 성능을 위해 SKILL.md 본문은 500줄 미만으로 유지하세요
* 이 한도에 가까워지면 콘텐츠를 별도의 파일로 분리하세요
* 아래 패턴을 사용하여 지침, 코드 및 리소스를 효과적으로 구성하세요

#### 시각적 개요: 단순한 구조에서 복잡한 구조로

기본적인 스킬은 메타데이터와 지침이 포함된 SKILL.md 파일 하나로 시작합니다:

<img src="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=87782ff239b297d9a9e8e1b72ed72db9" alt="Simple SKILL.md file showing YAML frontmatter and markdown body" data-og-width="2048" width="2048" data-og-height="1153" height="1153" data-path="images/agent-skills-simple-file.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=280&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=c61cc33b6f5855809907f7fda94cd80e 280w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=560&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=90d2c0c1c76b36e8d485f49e0810dbfd 560w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=840&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=ad17d231ac7b0bea7e5b4d58fb4aeabb 840w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=1100&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=f5d0a7a3c668435bb0aee9a3a8f8c329 1100w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=1650&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=0e927c1af9de5799cfe557d12249f6e6 1650w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=2500&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=46bbb1a51dd4c8202a470ac8c80a893d 2500w" />

스킬이 커지면 에이전트가 필요할 때만 로드하는 추가 콘텐츠를 번들로 묶을 수 있습니다:

<img src="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=a5e0aa41e3d53985a7e3e43668a33ea3" alt="Bundling additional reference files like reference.md and forms.md." data-og-width="2048" width="2048" data-og-height="1327" height="1327" data-path="images/agent-skills-bundling-content.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=280&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=f8a0e73783e99b4a643d79eac86b70a2 280w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=560&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=dc510a2a9d3f14359416b706f067904a 560w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=840&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=82cd6286c966303f7dd914c28170e385 840w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=1100&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=56f3be36c77e4fe4b523df209a6824c6 1100w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=1650&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=d22b5161b2075656417d56f41a74f3dd 1650w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=2500&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=3dd4bdd6850ffcc96c6c45fcb0acd6eb 2500w" />

완전한 스킬 디렉토리 구조는 다음과 같은 형태가 될 수 있습니다:

```
pdf/
├── SKILL.md              # 주요 지침 (트리거될 때 로드됨)
├── FORMS.md              # 양식 작성 가이드 (필요시 로드됨)
├── reference.md          # API 레퍼런스 (필요시 로드됨)
├── examples.md           # 사용 예시 (필요시 로드됨)
└── scripts/
    ├── analyze_form.py   # 유틸리티 스크립트 (로드되지 않고 실행됨)
    ├── fill_form.py      # 양식 작성 스크립트
    └── validate.py       # 검증 스크립트
```

#### 패턴 1: 레퍼런스가 포함된 상위 수준 가이드

````markdown  theme={null}
---
name: PDF Processing
description: Extracts text and tables from PDF files, fills forms, and merges documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
---

# PDF Processing

## 빠른 시작 (Quick start)

pdfplumber로 텍스트 추출하기:
```python
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

## 고급 기능 (Advanced features)

**양식 작성**: 전체 가이드는 [FORMS.md](FORMS.md)를 참조하세요
**API 레퍼런스**: 모든 메서드는 [REFERENCE.md](REFERENCE.md)를 참조하세요
**예시**: 일반적인 패턴은 [EXAMPLES.md](EXAMPLES.md)를 참조하세요
````

에이전트는 필요할 때만 FORMS.md, REFERENCE.md 또는 EXAMPLES.md를 로드합니다.

#### 패턴 2: 도메인별 구성

여러 도메인을 다루는 스킬의 경우, 관련 없는 컨텍스트를 로드하지 않도록 도메인별로 콘텐츠를 구성하세요. 사용자가 영업 지표에 대해 질문할 때 에이전트는 재무나 마케팅 데이터가 아닌 영업 관련 스키마만 읽으면 됩니다. 이렇게 하면 토큰 사용량을 낮추고 컨텍스트를 집중시킬 수 있습니다.

```
bigquery-skill/
├── SKILL.md (개요 및 탐색 안내)
└── reference/
    ├── finance.md (매출, 청구 지표)
    ├── sales.md (영업 기회, 파이프라인)
    ├── product.md (API 사용량, 기능)
    └── marketing.md (캠페인, 기여도 분석)
```

````markdown SKILL.md theme={null}
# BigQuery 데이터 분석

## 사용 가능한 데이터셋

**재무(Finance)**: 매출, ARR, 청구 → [reference/finance.md](reference/finance.md) 참조
**영업(Sales)**: 영업 기회, 파이프라인, 계정 → [reference/sales.md](reference/sales.md) 참조
**제품(Product)**: API 사용량, 기능, 도입률 → [reference/product.md](reference/product.md) 참조
**마케팅(Marketing)**: 캠페인, 기여도, 이메일 → [reference/marketing.md](reference/marketing.md) 참조

## 빠른 검색

grep을 사용하여 특정 지표 찾기:

```bash
grep -i "revenue" reference/finance.md
grep -i "pipeline" reference/sales.md
grep -i "api usage" reference/product.md
```
````

#### 패턴 3: 조건부 세부 정보

기본 콘텐츠를 보여주고, 고급 콘텐츠는 링크로 연결합니다:

```markdown  theme={null}
# DOCX 처리

## 문서 생성

새 문서를 만들 때는 docx-js를 사용하세요. [DOCX-JS.md](DOCX-JS.md)를 참조하세요.

## 문서 편집

간단한 편집의 경우 XML을 직접 수정하세요.

**변경 내용 추적(Tracked changes)의 경우**: [REDLINING.md](REDLINING.md) 참조
**OOXML 세부 정보의 경우**: [OOXML.md](OOXML.md) 참조
```

에이전트는 사용자가 해당 기능을 필요로 할 때만 REDLINING.md 또는 OOXML.md를 읽습니다.

### 깊게 중첩된 참조 피하기

에이전트는 참조된 파일에서 또다시 참조된 파일을 만날 때 파일을 부분적으로만 읽을 수 있습니다. 중첩된 참조를 만나면 에이전트는 전체 파일을 읽는 대신 `head -100` 같은 명령어를 사용하여 내용을 미리 보기만 할 수 있으며, 이로 인해 불완전한 정보를 얻게 됩니다.

**참조는 SKILL.md에서 한 단계(one level) 깊이로만 유지하세요**. 에이전트가 필요할 때 전체 파일을 온전히 읽을 수 있도록 모든 레퍼런스 파일은 SKILL.md에서 직접 링크되어야 합니다.

**나쁜 예시: 너무 깊음**:

```markdown  theme={null}
# SKILL.md
[advanced.md](advanced.md)를 참조하세요...

# advanced.md
[details.md](details.md)를 참조하세요...

# details.md
여기에 실제 정보가 있습니다...
```

**좋은 예시: 한 단계 깊이**:

```markdown  theme={null}
# SKILL.md

**기본 사용법**: [SKILL.md 내의 지침]
**고급 기능**: [advanced.md](advanced.md) 참조
**API 레퍼런스**: [reference.md](reference.md) 참조
**예시**: [examples.md](examples.md) 참조
```

### 긴 레퍼런스 파일은 목차로 구조화하기

100줄이 넘는 레퍼런스 파일의 경우 상단에 목차(table of contents)를 포함하세요. 이렇게 하면 에이전트가 부분 읽기로 미리 보기를 할 때에도 사용 가능한 정보의 전체 범위를 파악할 수 있습니다.

**예시**:

```markdown  theme={null}
# API 레퍼런스

## 목차 (Contents)
- 인증 및 설정
- 핵심 메서드 (생성, 조회, 수정, 삭제)
- 고급 기능 (배치 작업, 웹훅)
- 에러 처리 패턴
- 코드 예시

## 인증 및 설정
...

## 핵심 메서드
...
```

그런 다음 에이전트는 필요에 따라 전체 파일을 읽거나 특정 섹션으로 바로 이동할 수 있습니다.

이러한 파일시스템 기반 아키텍처가 어떻게 점진적 공개를 가능하게 하는지에 대한 자세한 내용은 아래 고급 섹션의 [런타임 환경(#runtime-environment)](#runtime-environment) 섹션을 참조하세요.

## 워크플로우 및 피드백 루프

### 복잡한 작업에는 워크플로우 사용하기

복잡한 작업은 명확하고 순차적인 단계로 나누세요. 특히 복잡한 워크플로우의 경우, 에이전트가 응답에 복사해 넣고 진행하면서 하나씩 체크할 수 있는 체크리스트를 제공하세요.

**예시 1: 리서치 종합 워크플로우** (코드가 없는 스킬의 경우):

````markdown  theme={null}
## 리서치 종합 워크플로우

이 체크리스트를 복사하고 진행 상황을 추적하세요:

```
리서치 진행 상황 (Research Progress):
- [ ] 1단계: 모든 원본 문서 읽기
- [ ] 2단계: 핵심 주제 파악하기
- [ ] 3단계: 주장 교차 검증하기
- [ ] 4단계: 구조화된 요약 작성하기
- [ ] 5단계: 인용 출처 검증하기
```

**1단계: 모든 원본 문서 읽기**

`sources/` 디렉토리에 있는 각 문서를 검토합니다. 주요 주장과 뒷받침하는 근거를 기록하세요.

**2단계: 핵심 주제 파악하기**

여러 출처에 걸친 패턴을 찾습니다. 어떤 주제가 반복적으로 나타납니까? 출처들이 동의하거나 의견이 갈리는 지점은 어디입니까?

**3단계: 주장 교차 검증하기**

각 주요 주장이 원본 자료에 실제로 나타나는지 확인합니다. 각 요점을 어떤 출처가 뒷받침하는지 기록하세요.

**4단계: 구조화된 요약 작성하기**

발견한 내용을 주제별로 정리합니다. 다음 내용을 포함하세요:
- 주요 주장
- 출처에서 가져온 뒷받침 근거
- 상충되는 관점 (있는 경우)

**5단계: 인용 출처 검증하기**

모든 주장이 올바른 원본 문서를 참조하고 있는지 확인합니다. 인용이 불완전한 경우 3단계로 돌아가세요.
````

이 예시는 코드가 필요하지 않은 분석 작업에 워크플로우가 어떻게 적용되는지 보여줍니다. 체크리스트 패턴은 복잡하고 여러 단계로 이루어진 모든 프로세스에 효과적입니다.

**예시 2: PDF 양식 작성 워크플로우** (코드가 포함된 스킬의 경우):

````markdown  theme={null}
## PDF 양식 작성 워크플로우

이 체크리스트를 복사하고 완료할 때마다 항목을 체크하세요:

```
작업 진행 상황 (Task Progress):
- [ ] 1단계: 양식 분석하기 (analyze_form.py 실행)
- [ ] 2단계: 필드 매핑 생성하기 (fields.json 편집)
- [ ] 3단계: 매핑 검증하기 (validate_fields.py 실행)
- [ ] 4단계: 양식 채우기 (fill_form.py 실행)
- [ ] 5단계: 출력 검증하기 (verify_output.py 실행)
```

**1단계: 양식 분석하기**

실행: `python scripts/analyze_form.py input.pdf`

이 스크립트는 양식 필드와 그 위치를 추출하여 `fields.json`에 저장합니다.

**2단계: 필드 매핑 생성하기**

`fields.json`을 편집하여 각 필드에 대한 값을 추가합니다.

**3단계: 매핑 검증하기**

실행: `python scripts/validate_fields.py fields.json`

계속 진행하기 전에 모든 검증 오류를 수정하세요.

**4단계: 양식 채우기**

실행: `python scripts/fill_form.py input.pdf fields.json output.pdf`

**5단계: 출력 검증하기**

실행: `python scripts/verify_output.py output.pdf`

검증에 실패하면 2단계로 돌아가세요.
````

명확한 단계는 에이전트가 중요한 검증 과정을 건너뛰는 것을 방지합니다. 체크리스트는 사용자와 에이전트 모두가 다단계 워크플로우의 진행 상황을 추적하는 데 도움이 됩니다.

### 피드백 루프 구현하기

**일반적인 패턴**: 검증기(validator) 실행 → 오류 수정 → 반복

이 패턴은 출력 품질을 크게 향상시킵니다.

**예시 1: 스타일 가이드 준수** (코드가 없는 스킬의 경우):

```markdown  theme={null}
## 콘텐츠 리뷰 프로세스

1. STYLE_GUIDE.md의 가이드라인에 따라 콘텐츠 초안을 작성합니다
2. 체크리스트에 따라 검토합니다:
   - 용어 일관성 확인
   - 예시가 표준 형식을 따르는지 검증
   - 모든 필수 섹션이 포함되어 있는지 확인
3. 문제가 발견된 경우:
   - 구체적인 섹션 참조와 함께 각 문제를 기록합니다
   - 콘텐츠를 수정합니다
   - 체크리스트를 다시 검토합니다
4. 모든 요구 사항이 충족되었을 때만 다음으로 진행합니다
5. 문서를 최종 확정하고 저장합니다
```

이는 스크립트 대신 레퍼런스 문서를 사용하는 검증 루프 패턴을 보여줍니다. 여기서 "검증기"는 STYLE\_GUIDE.md이며, 에이전트는 문서를 읽고 비교하여 검사를 수행합니다.

**예시 2: 문서 편집 프로세스** (코드가 포함된 스킬의 경우):

```markdown  theme={null}
## 문서 편집 프로세스

1. `word/document.xml`을 편집합니다
2. **즉시 검증합니다**: `python ooxml/scripts/validate.py unpacked_dir/`
3. 검증에 실패한 경우:
   - 에러 메시지를 주의 깊게 확인합니다
   - XML의 문제를 수정합니다
   - 검증을 다시 실행합니다
4. **검증을 통과했을 때만 다음으로 진행합니다**
5. 다시 패키징합니다: `python ooxml/scripts/pack.py unpacked_dir/ output.docx`
6. 출력 문서를 테스트합니다
```

검증 루프는 오류를 조기에 잡아냅니다.

## 콘텐츠 가이드라인

### 시점에 민감한(Time-sensitive) 정보 피하기

시간이 지나면 구식이 될 정보를 포함하지 마세요:

**나쁜 예시: 시점에 민감함** (나중에 틀린 정보가 됨):

```markdown  theme={null}
2025년 8월 이전에 이 작업을 수행하는 경우 구버전 API를 사용하세요.
2025년 8월 이후에는 신버전 API를 사용하세요.
```

**좋은 예시** ("과거 패턴" 섹션 사용):

```markdown  theme={null}
## 현재 방식 (Current method)

v2 API 엔드포인트를 사용하세요: `api.example.com/v2/messages`

## 과거 패턴 (Old patterns)

<details>
<summary>레거시 v1 API (2025-08 지원 중단됨)</summary>

v1 API는 다음을 사용했습니다: `api.example.com/v1/messages`

이 엔드포인트는 더 이상 지원되지 않습니다.
</details>
```

과거 패턴 섹션은 메인 콘텐츠를 어지럽히지 않으면서 역사적 컨텍스트를 제공합니다.

### 일관된 용어 사용하기

하나의 용어를 선택하여 스킬 전체에서 일관되게 사용하세요:

**좋음 - 일관됨**:

* 항상 "API 엔드포인트(API endpoint)" 사용
* 항상 "필드(field)" 사용
* 항상 "추출(extract)" 사용

**나쁨 - 일관성 없음**:

* "API 엔드포인트", "URL", "API 라우트", "경로(path)"를 혼용
* "필드", "박스", "엘리먼트", "컨트롤"을 혼용
* "추출하다(extract)", "가져오다(pull)", "얻다(get)", "조회하다(retrieve)"를 혼용

일관성은 에이전트가 지침을 이해하고 따르는 데 도움이 됩니다.

## 일반적인 패턴

### 템플릿 패턴

출력 형식에 대한 템플릿을 제공하세요. 필요에 따라 엄격함의 수준을 조절하세요.

**엄격한 요구 사항의 경우** (API 응답이나 데이터 형식 등):

````markdown  theme={null}
## 리포트 구조

항상(ALWAYS) 아래의 정확한 템플릿 구조를 사용하세요:

```markdown
# [분석 제목]

## 핵심 요약 (Executive summary)
[주요 발견 사항에 대한 한 문단 개요]

## 주요 발견 사항 (Key findings)
- 뒷받침하는 데이터가 포함된 발견 사항 1
- 뒷받침하는 데이터가 포함된 발견 사항 2
- 뒷받침하는 데이터가 포함된 발견 사항 3

## 권장 사항 (Recommendations)
1. 구체적이고 실행 가능한 권장 사항
2. 구체적이고 실행 가능한 권장 사항
```
````

**유연한 가이드의 경우** (상황에 맞는 조정이 유용할 때):

````markdown  theme={null}
## 리포트 구조

다음은 합리적인 기본 형식이나, 분석 내용에 따라 최선의 판단을 내리세요:

```markdown
# [분석 제목]

## 핵심 요약 (Executive summary)
[개요]

## 주요 발견 사항 (Key findings)
[발견한 내용에 따라 섹션을 조정하세요]

## 권장 사항 (Recommendations)
[구체적인 컨텍스트에 맞게 작성하세요]
```

특정 분석 유형에 맞게 필요에 따라 섹션을 조정하세요.
````

### 예시(Examples) 패턴

예시를 보는 것에 따라 출력 품질이 좌우되는 스킬의 경우, 일반적인 프롬프팅에서처럼 입력/출력 쌍을 제공하세요:

````markdown  theme={null}
## 커밋 메시지 형식

다음 예시를 따라 커밋 메시지를 생성하세요:

**예시 1:**
입력: Added user authentication with JWT tokens
출력:
```
feat(auth): implement JWT-based authentication

Add login endpoint and token validation middleware
```

**예시 2:**
입력: Fixed bug where dates displayed incorrectly in reports
출력:
```
fix(reports): correct date formatting in timezone conversion

Use UTC timestamps consistently across report generation
```

**예시 3:**
입력: Updated dependencies and refactored error handling
출력:
```
chore: update dependencies and refactor error handling

- Upgrade lodash to 4.17.21
- Standardize error response format across endpoints
```

이 스타일을 따르세요: type(scope): 간단한 설명, 그 다음에 상세한 설명.
````

예시는 단순한 설명만 제공할 때보다 에이전트가 원하는 스타일과 세부 수준을 더 명확하게 이해하도록 돕습니다.

### 조건부 워크플로우 패턴

에이전트가 의사결정 지점을 통과하도록 안내하세요:

```markdown  theme={null}
## 문서 수정 워크플로우

1. 수정 유형을 결정합니다:

   **새로운 콘텐츠를 생성합니까?** → 아래의 "생성 워크플로우"를 따르세요
   **기존 콘텐츠를 편집합니까?** → 아래의 "편집 워크플로우"를 따르세요

2. 생성 워크플로우:
   - docx-js 라이브러리 사용
   - 처음부터 문서 구축
   - .docx 형식으로 내보내기

3. 편집 워크플로우:
   - 기존 문서 압축 해제(unpack)
   - XML 직접 수정
   - 각 변경 후 검증
   - 완료 시 다시 패키징(repack)
```

<Tip>
  워크플로우가 커지거나 단계가 많아 복잡해지면 별도의 파일로 분리하고, 에이전트에게 당면한 작업에 따라 적절한 파일을 읽도록 지시하는 것을 고려하세요.
</Tip>

## 평가 및 반복 개선 (Evaluation and iteration)

### 평가를 먼저 구축하기

**방대한 문서를 작성하기 전에 평가(evaluation)를 먼저 만드세요.** 이렇게 하면 상상 속의 문제를 문서화하는 대신 스킬이 실제 문제를 해결하도록 보장할 수 있습니다.

**평가 주도 개발 (Evaluation-driven development):**

1. **격차(Gap) 파악하기**: 스킬 없이 대표적인 작업들에 대해 에이전트를 실행합니다. 구체적인 실패 사례나 누락된 컨텍스트를 기록합니다
2. **평가 만들기**: 이러한 격차를 테스트하는 세 가지 시나리오를 구축합니다
3. **베이스라인(Baseline) 설정하기**: 스킬이 없을 때의 에이전트 성능을 측정합니다
4. **최소한의 지침 작성하기**: 격차를 해소하고 평가를 통과하기에 딱 충분한 만큼의 콘텐츠만 작성합니다
5. **반복 개선하기(Iterate)**: 평가를 실행하고, 베이스라인과 비교하며, 다듬어 나갑니다

이 접근 방식은 결코 발생하지 않을 수도 있는 요구 사항을 미리 예측하는 대신 실제 문제를 해결하도록 보장합니다.

**평가 구조**:

```json  theme={null}
{
  "skills": ["pdf-processing"],
  "query": "이 PDF 파일에서 모든 텍스트를 추출하여 output.txt에 저장해 줘",
  "files": ["test-files/document.pdf"],
  "expected_behavior": [
    "적절한 PDF 처리 라이브러리 또는 커맨드라인 도구를 사용하여 PDF 파일을 성공적으로 읽는다",
    "누락되는 페이지 없이 문서의 모든 페이지에서 텍스트 콘텐츠를 추출한다",
    "추출된 텍스트를 명확하고 읽기 쉬운 형식으로 output.txt라는 이름의 파일에 저장한다"
  ]
}
```

<Note>
  이 예시는 간단한 테스트 루브릭을 사용한 데이터 기반 평가를 보여줍니다. 현재 이러한 평가를 실행하는 내장 기능은 제공하지 않습니다. 사용자는 자체 평가 시스템을 구축할 수 있습니다. 평가는 스킬의 효과를 측정하는 진실의 원천(source of truth)입니다.
</Note>

### 에이전트와 함께 반복적으로 스킬 개발하기

가장 효과적인 스킬 개발 프로세스는 에이전트 자체를 참여시키는 것입니다. 하나의 인스턴스("에이전트 A")와 협력하여 다른 인스턴스("에이전트 B")가 사용할 스킬을 만드세요. 에이전트 A는 지침을 설계하고 다듬는 것을 돕고, 에이전트 B는 실제 작업에서 이를 테스트합니다. 기반 모델은 효과적인 에이전트 지침을 작성하는 방법과 에이전트에게 어떤 정보가 필요한지를 모두 이해하고 있기 때문에 이 방식이 잘 작동합니다.

**새로운 스킬 만들기:**

1. **스킬 없이 작업 완료하기**: 일반적인 프롬프팅을 사용하여 에이전트 A와 함께 문제를 해결해 나갑니다. 작업하면서 자연스럽게 컨텍스트를 제공하고, 선호도를 설명하며, 절차적 지식을 공유하게 됩니다. 어떤 정보를 반복적으로 제공하는지 주목하세요.

2. **재사용 가능한 패턴 파악하기**: 작업을 완료한 후, 향후 유사한 작업에 유용할 만한 어떤 컨텍스트를 제공했는지 파악합니다.

   **예시**: BigQuery 분석을 진행했다면 테이블 이름, 필드 정의, 필터링 규칙("항상 테스트 계정은 제외할 것" 등), 일반적인 쿼리 패턴을 제공했을 수 있습니다.

3. **에이전트 A에게 스킬 생성 요청하기**: "방금 우리가 사용한 이 BigQuery 분석 패턴을 담은 스킬을 만들어 줘. 테이블 스키마, 명명 규칙, 그리고 테스트 계정을 필터링하는 규칙을 포함해 줘."

   <Tip>
     최신 에이전트는 스킬 형식과 구조를 기본적으로(natively) 이해합니다. 스킬 작성을 돕기 위해 특별한 시스템 프롬프트나 "writing skills" 스킬이 꼭 필요한 것은 아닙니다. 에이전트에게 스킬을 만들어 달라고 요청하기만 하면 적절한 프론트매터와 본문 내용을 갖춘 올바른 구조의 SKILL.md 콘텐츠를 생성합니다.
   </Tip>

4. **간결성 검토하기**: 에이전트 A가 불필요한 설명을 추가하지 않았는지 확인합니다. 다음과 같이 요청하세요: "승률(win rate)이 무엇을 의미하는지에 대한 설명은 제거해 줘. 에이전트는 이미 그걸 알고 있어."

5. **정보 아키텍처 개선하기**: 에이전트 A에게 콘텐츠를 더 효과적으로 구성하도록 요청합니다. 예: "테이블 스키마가 별도의 레퍼런스 파일에 들어가도록 정리해 줘. 나중에 테이블을 더 추가할 수도 있어."

6. **유사한 작업에서 테스트하기**: 관련된 사용 사례에서 에이전트 B(스킬이 로드된 새로운 인스턴스)와 함께 스킬을 사용해 봅니다. 에이전트 B가 올바른 정보를 찾고, 규칙을 정확히 적용하며, 작업을 성공적으로 처리하는지 관찰합니다.

7. **관찰 결과를 바탕으로 반복 개선하기**: 에이전트 B가 어려움을 겪거나 무언가를 놓치면 구체적인 내용을 가지고 에이전트 A에게 돌아갑니다: "에이전트가 이 스킬을 사용했을 때 4분기(Q4) 날짜로 필터링하는 것을 깜빡했어. 날짜 필터링 패턴에 대한 섹션을 추가해야 할까?"

**기존 스킬 반복 개선하기:**

스킬을 개선할 때에도 동일한 계층적 패턴이 계속됩니다. 다음 과정을 번갈아 수행합니다:

* **에이전트 A와 작업하기** (스킬을 다듬는 것을 돕는 전문가)
* **에이전트 B로 테스트하기** (스킬을 사용하여 실제 작업을 수행하는 에이전트)
* **에이전트 B의 행동 관찰하기** 및 에이전트 A에게 인사이트 가져가기

1. **실제 워크플로우에서 스킬 사용하기**: 에이전트 B(스킬이 로드된 상태)에게 테스트 시나리오가 아닌 실제 작업을 부여합니다

2. **에이전트 B의 행동 관찰하기**: 어디서 어려움을 겪는지, 어디서 성공하는지, 또는 예상치 못한 선택을 하는지 기록합니다

   **관찰 예시**: "에이전트 B에게 지역별 영업 리포트를 요청했을 때, 스킬에 이 규칙이 언급되어 있음에도 불구하고 쿼리를 작성하면서 테스트 계정을 필터링해 제외하는 것을 잊어버렸다."

3. **개선을 위해 에이전트 A에게 돌아가기**: 현재의 SKILL.md를 공유하고 관찰한 내용을 설명합니다. 질문하기: "지역별 리포트를 요청했을 때 에이전트 B가 테스트 계정을 필터링하는 것을 잊어버리는 걸 봤어. 스킬에 필터링이 언급되어 있긴 한데, 눈에 잘 띄지 않는 걸까?"

4. **에이전트 A의 제안 검토하기**: 에이전트 A는 규칙이 더 눈에 띄도록 재구성하거나, "always filter" 대신 "MUST filter" 같은 더 강력한 표현을 사용하거나, 워크플로우 섹션을 재구조화할 것을 제안할 수 있습니다.

5. **변경 사항 적용 및 테스트하기**: 에이전트 A가 다듬은 내용으로 스킬을 업데이트한 다음, 유사한 요청에 대해 에이전트 B로 다시 테스트합니다

6. **사용에 따라 반복하기**: 새로운 시나리오를 마주칠 때마다 이 관찰-다듬기-테스트(observe-refine-test) 사이클을 계속합니다. 각 반복은 추측이 아닌 실제 에이전트 행동을 기반으로 스킬을 개선합니다.

**팀 피드백 수집하기:**

1. 팀원들과 스킬을 공유하고 그들의 사용 방식을 관찰합니다
2. 질문하기: 예상할 때 스킬이 활성화됩니까? 지침이 명확합니까? 무엇이 빠져 있습니까?
3. 피드백을 반영하여 자신의 사용 패턴에서 놓쳤던 사각지대를 해소합니다

**이 접근 방식이 작동하는 이유**: 에이전트 A는 에이전트의 필요를 이해하고, 사용자는 도메인 전문 지식을 제공하며, 에이전트 B는 실제 사용을 통해 격차를 드러내고, 반복적인 개선은 가정이 아닌 관찰된 행동을 바탕으로 스킬을 향상시킵니다.

### 에이전트가 스킬을 탐색하는 방식 관찰하기

스킬을 반복 개선할 때 에이전트가 실제로 스킬을 어떻게 사용하는지 주의를 기울이세요. 다음 사항들을 주의 깊게 살펴보세요:

* **예상치 못한 탐색 경로**: 에이전트가 예상하지 못한 순서로 파일을 읽습니까? 이는 구조가 생각만큼 직관적이지 않다는 신호일 수 있습니다
* **놓친 연결 고리**: 에이전트가 중요한 파일에 대한 참조를 따라가는 데 실패합니까? 링크가 더 명시적이거나 눈에 잘 띄어야 할 수 있습니다
* **특정 섹션에 대한 과도한 의존**: 에이전트가 동일한 파일을 반복해서 읽는다면, 해당 콘텐츠가 메인 SKILL.md에 있어야 하는지 고려해 보세요
* **무시된 콘텐츠**: 에이전트가 번들로 묶인 파일에 전혀 접근하지 않는다면, 그 파일은 불필요하거나 메인 지침에서 신호가 제대로 주어지지 않은 것일 수 있습니다

가정이 아닌 이러한 관찰 결과를 바탕으로 반복 개선하세요. 스킬 메타데이터의 'name'과 'description'은 특히 중요합니다. 에이전트는 현재 작업에 응답하여 스킬을 트리거할지 여부를 결정할 때 이 정보들을 사용합니다. 스킬이 무엇을 하는지와 언제 사용되어야 하는지를 명확하게 설명하는지 확인하세요.

## 피해야 할 안티패턴

### Windows 스타일 경로 피하기

Windows 환경이더라도 파일 경로에는 항상 슬래시(forward slash, `/`)를 사용하세요:

* ✓ **좋음**: `scripts/helper.py`, `reference/guide.md`
* ✗ **피해야 함**: `scripts\helper.py`, `reference\guide.md`

Unix 스타일 경로는 모든 플랫폼에서 작동하지만, Windows 스타일 경로는 Unix 시스템에서 오류를 발생시킵니다.

### 너무 많은 선택지 제공 피하기

꼭 필요한 경우가 아니라면 여러 접근 방식을 나열하지 마세요:

````markdown  theme={null}
**나쁜 예시: 너무 많은 선택지** (혼란스러움):
"pypdf, 또는 pdfplumber, 또는 PyMuPDF, 또는 pdf2image 등을 사용할 수 있습니다..."

**좋은 예시: 기본값(default) 제공하기** (예외 상황을 위한 탈출구 포함):
"텍스트 추출에는 pdfplumber를 사용하세요:
```python
import pdfplumber
```

OCR이 필요한 스캔된 PDF의 경우, 대신 pytesseract와 함께 pdf2image를 사용하세요."
````

## 고급: 실행 가능한 코드가 포함된 스킬

아래 섹션들은 실행 가능한 스크립트를 포함하는 스킬에 초점을 맞춥니다. 스킬이 마크다운 지침만 사용하는 경우 [효과적인 스킬을 위한 체크리스트](#checklist-for-effective-skills)로 건너뛰세요.

### 에이전트에게 떠넘기지 말고 직접 해결하기 (Solve, don't punt)

스킬용 스크립트를 작성할 때는 에러 상황을 에이전트에게 떠넘기지(punting) 말고 직접 처리하세요.

**좋은 예시: 에러를 명시적으로 처리하기**:

```python  theme={null}
def process_file(path):
    """파일을 처리하며, 파일이 존재하지 않으면 새로 생성합니다."""
    try:
        with open(path) as f:
            return f.read()
    except FileNotFoundError:
        # 실패하는 대신 기본 콘텐츠로 파일 생성
        print(f"File {path} not found, creating default")
        with open(path, 'w') as f:
            f.write('')
        return ''
    except PermissionError:
        # 실패하는 대신 대안 제공
        print(f"Cannot access {path}, using default")
        return ''
```

**나쁜 예시: 에이전트에게 떠넘기기**:

```python  theme={null}
def process_file(path):
    # 그냥 실패하게 두고 에이전트가 알아서 해결하게 함
    return open(path).read()
```

또한 설정 매개변수들은 "부두 상수(voodoo constants)"(Ousterhout의 법칙)를 피하기 위해 그 근거가 설명되고 문서화되어야 합니다. 작성자조차 올바른 값을 모른다면 에이전트가 어떻게 그것을 결정하겠습니까?

**좋은 예시: 자체 문서화(Self-documenting)**:

```python  theme={null}
# HTTP 요청은 일반적으로 30초 이내에 완료됩니다
# 더 긴 타임아웃은 느린 연결을 고려한 것입니다
REQUEST_TIMEOUT = 30

# 3회의 재시도는 안정성과 속도 사이의 균형을 맞춥니다
# 대부분의 간헐적 실패는 두 번째 재시도까지 해결됩니다
MAX_RETRIES = 3
```

**나쁜 예시: 매직 넘버(Magic numbers)**:

```python  theme={null}
TIMEOUT = 47  # 왜 47인가?
RETRIES = 5   # 왜 5인가?
```

### 유틸리티 스크립트 제공하기

에이전트가 스크립트를 직접 작성할 수 있더라도, 미리 만들어진 스크립트는 여러 이점을 제공합니다:

**유틸리티 스크립트의 이점**:

* 생성된 코드보다 더 신뢰할 수 있음
* 토큰 절약 (컨텍스트에 코드를 포함할 필요 없음)
* 시간 절약 (코드 생성이 필요 없음)
* 여러 번 사용해도 일관성 보장

<img src="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=4bbc45f2c2e0bee9f2f0d5da669bad00" alt="Bundling executable scripts alongside instruction files" data-og-width="2048" width="2048" data-og-height="1154" height="1154" data-path="images/agent-skills-executable-scripts.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=280&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=9a04e6535a8467bfeea492e517de389f 280w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=560&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=e49333ad90141af17c0d7651cca7216b 560w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=840&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=954265a5df52223d6572b6214168c428 840w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=1100&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=2ff7a2d8f2a83ee8af132b29f10150fd 1100w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=1650&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=48ab96245e04077f4d15e9170e081cfb 1650w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=2500&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=0301a6c8b3ee879497cc5b5483177c90 2500w" />

위 다이어그램은 실행 가능한 스크립트가 지침 파일과 함께 어떻게 작동하는지 보여줍니다. 지침 파일(forms.md)이 스크립트를 참조하면, 에이전트는 스크립트의 내용을 컨텍스트에 로드하지 않고도 이를 실행할 수 있습니다.

**중요한 구분**: 지침에서 에이전트가 다음 중 무엇을 해야 하는지 명확히 밝히세요:

* **스크립트 실행** (가장 일반적): "필드를 추출하려면 `analyze_form.py`를 실행하세요"
* **레퍼런스로 읽기** (복잡한 로직의 경우): "필드 추출 알고리즘은 `analyze_form.py`를 참조하세요"

대부분의 유틸리티 스크립트의 경우 더 신뢰할 수 있고 효율적이므로 실행하는 것이 선호됩니다. 스크립트 실행이 작동하는 방식에 대한 자세한 내용은 아래 [런타임 환경](#runtime-environment) 섹션을 참조하세요.

**예시**:

````markdown  theme={null}
## 유틸리티 스크립트

**analyze_form.py**: PDF에서 모든 양식 필드 추출

```bash
python scripts/analyze_form.py input.pdf > fields.json
```

출력 형식:
```json
{
  "field_name": {"type": "text", "x": 100, "y": 200},
  "signature": {"type": "sig", "x": 150, "y": 500}
}
```

**validate_boxes.py**: 겹치는 바운딩 박스(bounding boxes) 확인

```bash
python scripts/validate_boxes.py fields.json
# 반환값: "OK" 또는 충돌 목록 나열
```

**fill_form.py**: PDF에 필드 값 적용

```bash
python scripts/fill_form.py input.pdf fields.json output.pdf
```
````

### 시각적 분석 사용하기

입력을 이미지로 렌더링할 수 있는 경우, 에이전트가 이미지를 분석하게 하세요:

````markdown  theme={null}
## 양식 레이아웃 분석

1. PDF를 이미지로 변환합니다:
   ```bash
   python scripts/pdf_to_images.py form.pdf
   ```

2. 각 페이지 이미지를 분석하여 양식 필드를 식별합니다
3. 에이전트가 필드 위치와 유형을 시각적으로 확인할 수 있습니다
````

<Note>
  이 예시에서는 `pdf_to_images.py` 스크립트를 직접 작성해야 합니다.
</Note>

에이전트의 비전(vision) 기능은 레이아웃과 구조를 이해하는 데 도움이 됩니다.

### 검증 가능한 중간 출력물 만들기

에이전트가 복잡하고 개방형인(open-ended) 작업을 수행할 때 실수를 범할 수 있습니다. "계획-검증-실행(plan-validate-execute)" 패턴은 에이전트가 먼저 구조화된 형식으로 계획을 작성하게 한 다음, 이를 실행하기 전에 스크립트로 그 계획을 검증함으로써 오류를 조기에 잡아냅니다.

**예시**: 스프레드시트를 기반으로 PDF의 양식 필드 50개를 업데이트하라고 에이전트에게 요청한다고 상상해 보세요. 검증이 없으면 존재하지 않는 필드를 참조하거나, 충돌하는 값을 생성하거나, 필수 필드를 누락하거나, 업데이트를 잘못 적용할 수 있습니다.

**해결책**: 위에 표시된 워크플로우 패턴(PDF 양식 작성)을 사용하되, 변경 사항을 적용하기 전에 검증을 거치는 중간 파일인 `changes.json`을 추가합니다. 워크플로우는 다음과 같이 됩니다: 분석 → **계획 파일 생성** → **계획 검증** → 실행 → 검증.

**이 패턴이 작동하는 이유:**

* **오류 조기 포착**: 변경 사항이 적용되기 전에 검증 단계에서 문제를 찾아냅니다
* **기계적 검증 가능(Machine-verifiable)**: 스크립트가 객관적인 검증을 제공합니다
* **가역적인 계획 수립**: 에이전트가 원본을 건드리지 않고 계획을 반복해서 수정할 수 있습니다
* **명확한 디버깅**: 에러 메시지가 구체적인 문제를 정확히 짚어줍니다

**언제 사용해야 하는가**: 배치(batch) 작업, 파괴적인(destructive) 변경, 복잡한 검증 규칙, 위험도가 높은 작업.

**구현 팁**: 에이전트가 문제를 수정하는 데 도움이 되도록 "Field 'signature\_date' not found. Available fields: customer\_name, order\_total, signature\_date\_signed"와 같이 구체적인 에러 메시지를 포함하여 검증 스크립트가 상세한 출력을 내도록 만드세요.

### 패키지 의존성

스킬은 플랫폼별 제한 사항이 있는 코드 실행 환경에서 실행됩니다:

* **claude.ai**: npm 및 PyPI에서 패키지를 설치하고 GitHub 저장소에서 풀(pull)할 수 있습니다
* **Anthropic API**: 네트워크 액세스가 없으며 런타임 패키지 설치가 불가능합니다

SKILL.md에 필요한 패키지를 나열하고 [코드 실행 도구 문서(code execution tool documentation)](https://platform.claude.com/docs/en/agents-and-tools/tool-use/code-execution-tool)에서 해당 패키지들을 사용할 수 있는지 확인하세요.

### 런타임 환경 {#runtime-environment}

스킬은 파일시스템 액세스, bash 명령어 및 코드 실행 기능을 갖춘 코드 실행 환경에서 실행됩니다. 이 아키텍처에 대한 개념적 설명은 개요 문서의 [스킬 아키텍처(The Skills architecture)](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#the-skills-architecture)를 참조하세요.

**이것이 스킬 작성에 미치는 영향:**

**에이전트가 스킬에 접근하는 방식:**

1. **메타데이터 사전 로드**: 시작 시 모든 스킬의 YAML 프론트매터에서 `name`과 `description`이 시스템 프롬프트에 로드됩니다
2. **온디맨드(On-demand) 파일 읽기**: 에이전트는 필요할 때 파일 읽기 도구를 사용하여 파일시스템에서 SKILL.md 및 기타 파일에 접근합니다
3. **효율적인 스크립트 실행**: 유틸리티 스크립트는 전체 내용을 컨텍스트에 로드하지 않고도 bash를 통해 실행할 수 있습니다. 스크립트의 출력만 토큰을 소비합니다
4. **대용량 파일에 대한 컨텍스트 페널티 없음**: 레퍼런스 파일, 데이터 또는 문서는 실제로 읽히기 전까지 컨텍스트 토큰을 소비하지 않습니다

* **파일 경로가 중요합니다**: 에이전트는 스킬 디렉토리를 파일시스템처럼 탐색합니다. 백슬래시가 아닌 슬래시(`reference/guide.md`)를 사용하세요
* **파일 이름을 설명적으로 지정하세요**: 내용을 나타내는 이름을 사용하세요 (`doc2.md`가 아니라 `form_validation_rules.md`)
* **탐색하기 쉽게 구성하세요**: 도메인이나 기능별로 디렉토리를 구조화하세요
  * 좋음: `reference/finance.md`, `reference/sales.md`
  * 나쁨: `docs/file1.md`, `docs/file2.md`
* **포괄적인 리소스를 번들로 묶으세요**: 완전한 API 문서, 방대한 예시, 대규모 데이터셋을 포함하세요. 접근하기 전까지는 컨텍스트 페널티가 없습니다
* **결정론적인(deterministic) 작업에는 스크립트를 선호하세요**: 에이전트에게 검증 코드를 생성하라고 요청하는 대신 `validate_form.py`를 작성하세요
* **실행 의도를 명확히 하세요**:
  * "필드를 추출하려면 `analyze_form.py`를 실행하세요" (실행)
  * "추출 알고리즘은 `analyze_form.py`를 참조하세요" (레퍼런스로 읽기)
* **파일 접근 패턴을 테스트하세요**: 실제 요청으로 테스트하여 에이전트가 디렉토리 구조를 잘 탐색할 수 있는지 확인하세요

**예시:**

```
bigquery-skill/
├── SKILL.md (개요, 레퍼런스 파일들을 가리킴)
└── reference/
    ├── finance.md (매출 지표)
    ├── sales.md (파이프라인 데이터)
    └── product.md (사용량 분석)
```

사용자가 매출에 대해 질문하면, 에이전트는 SKILL.md를 읽고 `reference/finance.md`에 대한 참조를 확인한 뒤 bash를 호출하여 해당 파일만 읽습니다. `sales.md`와 `product.md` 파일은 파일시스템에 그대로 남아 필요할 때까지 컨텍스트 토큰을 전혀 소비하지 않습니다. 이러한 파일시스템 기반 모델이 바로 점진적 공개를 가능하게 하는 핵심입니다. 에이전트는 탐색을 통해 각 작업에 필요한 것만 선택적으로 로드할 수 있습니다.

기술 아키텍처에 대한 전체 세부 정보는 스킬 개요의 [스킬 작동 방식(How Skills work)](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#how-skills-work)을 참조하세요.

### MCP 도구 참조

스킬이 MCP(Model Context Protocol) 도구를 사용하는 경우, "tool not found" 오류를 방지하기 위해 항상 완전히 정규화된 도구 이름(fully qualified tool names)을 사용하세요.

**형식**: `ServerName:tool_name`

**예시**:

```markdown  theme={null}
테이블 스키마를 조회하려면 BigQuery:bigquery_schema 도구를 사용하세요.
이슈를 생성하려면 GitHub:create_issue 도구를 사용하세요.
```

여기서:

* `BigQuery`와 `GitHub`는 MCP 서버 이름입니다
* `bigquery_schema`와 `create_issue`는 해당 서버 내의 도구 이름입니다

서버 접두사가 없으면, 특히 여러 MCP 서버가 사용 가능한 환경에서 에이전트가 도구를 찾지 못할 수 있습니다.

### 도구가 설치되어 있다고 가정하지 않기

패키지가 당연히 사용 가능하다고 가정하지 마세요:

````markdown  theme={null}
**나쁜 예시: 설치되어 있다고 가정함**:
"pdf 라이브러리를 사용하여 파일을 처리하세요."

**좋은 예시: 의존성에 대해 명시적임**:
"필수 패키지를 설치하세요: `pip install pypdf`

그런 다음 사용하세요:
```python
from pypdf import PdfReader
reader = PdfReader("file.pdf")
```"
````

## 기술 노트

### YAML 프론트매터 요구 사항

SKILL.md 프론트매터에는 `name`(최대 64자) 및 `description`(최대 1024자) 필드가 필요합니다. 전체 구조 세부 정보는 [스킬 개요](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#skill-structure)를 참조하세요.

### 토큰 예산 (Token budgets)

최적의 성능을 위해 SKILL.md 본문은 500줄 미만으로 유지하세요. 콘텐츠가 이를 초과하는 경우 앞서 설명한 점진적 공개 패턴을 사용하여 별도의 파일로 분리하세요. 아키텍처 세부 정보는 [스킬 개요](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#how-skills-work)를 참조하세요.

## 효과적인 스킬을 위한 체크리스트 {#checklist-for-effective-skills}

스킬을 공유하기 전에 다음 사항들을 확인하세요:

### 핵심 품질

* [ ] 설명(description)이 구체적이며 핵심 용어를 포함하고 있는가
* [ ] 설명에 스킬이 무엇을 하는지와 언제 사용해야 하는지가 모두 포함되어 있는가
* [ ] SKILL.md 본문이 500줄 미만인가
* [ ] 추가 세부 정보가 별도의 파일로 분리되어 있는가 (필요한 경우)
* [ ] 시점에 민감한 정보가 없는가 (또는 "과거 패턴" 섹션에 분리되어 있는가)
* [ ] 전체적으로 용어가 일관되게 사용되었는가
* [ ] 예시가 추상적이지 않고 구체적인가
* [ ] 파일 참조가 한 단계(one level) 깊이로 유지되는가
* [ ] 점진적 공개가 적절하게 사용되었는가
* [ ] 워크플로우에 명확한 단계가 있는가

### 코드 및 스크립트

* [ ] 스크립트가 에이전트에게 떠넘기지 않고 문제를 직접 해결하는가
* [ ] 에러 처리가 명시적이고 도움이 되는가
* [ ] "부두 상수(voodoo constants)"가 없는가 (모든 값의 근거가 설명되어 있는가)
* [ ] 필수 패키지가 지침에 나열되어 있고 사용 가능한지 확인되었는가
* [ ] 스크립트에 명확한 문서화가 되어 있는가
* [ ] Windows 스타일 경로가 없는가 (모두 슬래시 `/` 사용)
* [ ] 중요한 작업에 대한 검증/확인 단계가 있는가
* [ ] 품질이 중요한 작업에 피드백 루프가 포함되어 있는가

### 테스트

* [ ] 최소 세 개의 평가(evaluation)가 생성되었는가
* [ ] Haiku, Sonnet, Opus에서 테스트되었는가
* [ ] 실제 사용 시나리오로 테스트되었는가
* [ ] 팀 피드백이 반영되었는가 (해당하는 경우)

## 다음 단계

<CardGroup cols={2}>
  <Card title="Agent Skills 시작하기" icon="rocket" href="https://platform.claude.com/docs/en/agents-and-tools/agent-skills/quickstart">
    첫 번째 스킬 만들기
  </Card>

  <Card title="Claude Code에서 스킬 사용하기" icon="terminal" href="https://code.claude.com/docs/en/skills">
    Claude Code에서 스킬 생성 및 관리하기
  </Card>

  <Card title="API로 스킬 사용하기" icon="code" href="https://platform.claude.com/docs/en/build-with-claude/skills-guide">
    프로그래밍 방식으로 스킬 업로드 및 사용하기
  </Card>
</CardGroup>
