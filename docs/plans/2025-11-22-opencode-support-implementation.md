# OpenCode 지원 구현 계획

> **에이전트 작업자를 위한 안내:** 필수 하위 스킬: 이 계획을 작업 단위로 구현하려면 `superpowers:executing-plans`를 사용하세요.

**목표:** 기존 Codex 구현과 핵심 기능을 공유하는 네이티브 JavaScript 플러그인을 통해 OpenCode.ai에 대한 전체 superpowers 지원을 추가합니다.

**아키텍처:** 공통 스킬 탐색/파싱 로직을 `lib/skills-core.js`로 추출하고, 이를 사용하도록 Codex를 리팩토링한 다음, 커스텀 도구와 세션 훅이 포함된 네이티브 플러그인 API를 사용하여 OpenCode 플러그인을 구축합니다.

**기술 스택:** Node.js, JavaScript, OpenCode Plugin API, Git worktrees

---

## 1단계: 공유 코어 모듈 생성

### 작업 1: 프론트매터(Frontmatter) 파싱 추출

**파일:**
- 생성: `lib/skills-core.js`
- 참조: `.codex/superpowers-codex` (40~74행)

**1단계: extractFrontmatter 함수가 포함된 lib/skills-core.js 생성**

```javascript
#!/usr/bin/env node

const fs = require('fs');
const path = require('path');

/**
 * Extract YAML frontmatter from a skill file.
 * Current format:
 * ---
 * name: skill-name
 * description: Use when [condition] - [what it does]
 * ---
 *
 * @param {string} filePath - Path to SKILL.md file
 * @returns {{name: string, description: string}}
 */
function extractFrontmatter(filePath) {
    try {
        const content = fs.readFileSync(filePath, 'utf8');
        const lines = content.split('\n');

        let inFrontmatter = false;
        let name = '';
        let description = '';

        for (const line of lines) {
            if (line.trim() === '---') {
                if (inFrontmatter) break;
                inFrontmatter = true;
                continue;
            }

            if (inFrontmatter) {
                const match = line.match(/^(\w+):\s*(.*)$/);
                if (match) {
                    const [, key, value] = match;
                    switch (key) {
                        case 'name':
                            name = value.trim();
                            break;
                        case 'description':
                            description = value.trim();
                            break;
                    }
                }
            }
        }

        return { name, description };
    } catch (error) {
        return { name: '', description: '' };
    }
}

module.exports = {
    extractFrontmatter
};
```

**2단계: 파일 생성 확인**

실행: `ls -l lib/skills-core.js`
예상 결과: 파일이 존재함

**3단계: 커밋**

```bash
git add lib/skills-core.js
git commit -m "feat: create shared skills core module with frontmatter parser"
```

---

### 작업 2: 스킬 탐색 로직 추출

**파일:**
- 수정: `lib/skills-core.js`
- 참조: `.codex/superpowers-codex` (97~136행)

**1단계: skills-core.js에 findSkillsInDir 함수 추가**

`module.exports` 앞에 다음을 추가합니다:

```javascript
/**
 * Find all SKILL.md files in a directory recursively.
 *
 * @param {string} dir - Directory to search
 * @param {string} sourceType - 'personal' or 'superpowers' for namespacing
 * @param {number} maxDepth - Maximum recursion depth (default: 3)
 * @returns {Array<{path: string, name: string, description: string, sourceType: string}>}
 */
function findSkillsInDir(dir, sourceType, maxDepth = 3) {
    const skills = [];

    if (!fs.existsSync(dir)) return skills;

    function recurse(currentDir, depth) {
        if (depth > maxDepth) return;

        const entries = fs.readdirSync(currentDir, { withFileTypes: true });

        for (const entry of entries) {
            const fullPath = path.join(currentDir, entry.name);

            if (entry.isDirectory()) {
                // Check for SKILL.md in this directory
                const skillFile = path.join(fullPath, 'SKILL.md');
                if (fs.existsSync(skillFile)) {
                    const { name, description } = extractFrontmatter(skillFile);
                    skills.push({
                        path: fullPath,
                        skillFile: skillFile,
                        name: name || entry.name,
                        description: description || '',
                        sourceType: sourceType
                    });
                }

                // Recurse into subdirectories
                recurse(fullPath, depth + 1);
            }
        }
    }

    recurse(dir, 0);
    return skills;
}
```

**2단계: module.exports 업데이트**

exports 구문을 다음으로 교체합니다:

```javascript
module.exports = {
    extractFrontmatter,
    findSkillsInDir
};
```

**3단계: 구문(Syntax) 확인**

실행: `node -c lib/skills-core.js`
예상 결과: 출력 없음 (성공)

**4단계: 커밋**

```bash
git add lib/skills-core.js
git commit -m "feat: add skill discovery function to core module"
```

---

### 작업 3: 스킬 경로 확인(Resolution) 로직 추출

**파일:**
- 수정: `lib/skills-core.js`
- 참조: `.codex/superpowers-codex` (212~280행)

**1단계: resolveSkillPath 함수 추가**

`module.exports` 앞에 다음을 추가합니다:

```javascript
/**
 * Resolve a skill name to its file path, handling shadowing
 * (personal skills override superpowers skills).
 *
 * @param {string} skillName - Name like "superpowers:brainstorming" or "my-skill"
 * @param {string} superpowersDir - Path to superpowers skills directory
 * @param {string} personalDir - Path to personal skills directory
 * @returns {{skillFile: string, sourceType: string, skillPath: string} | null}
 */
function resolveSkillPath(skillName, superpowersDir, personalDir) {
    // Strip superpowers: prefix if present
    const forceSuperpowers = skillName.startsWith('superpowers:');
    const actualSkillName = forceSuperpowers ? skillName.replace(/^superpowers:/, '') : skillName;

    // Try personal skills first (unless explicitly superpowers:)
    if (!forceSuperpowers && personalDir) {
        const personalPath = path.join(personalDir, actualSkillName);
        const personalSkillFile = path.join(personalPath, 'SKILL.md');
        if (fs.existsSync(personalSkillFile)) {
            return {
                skillFile: personalSkillFile,
                sourceType: 'personal',
                skillPath: actualSkillName
            };
        }
    }

    // Try superpowers skills
    if (superpowersDir) {
        const superpowersPath = path.join(superpowersDir, actualSkillName);
        const superpowersSkillFile = path.join(superpowersPath, 'SKILL.md');
        if (fs.existsSync(superpowersSkillFile)) {
            return {
                skillFile: superpowersSkillFile,
                sourceType: 'superpowers',
                skillPath: actualSkillName
            };
        }
    }

    return null;
}
```

**2단계: module.exports 업데이트**

```javascript
module.exports = {
    extractFrontmatter,
    findSkillsInDir,
    resolveSkillPath
};
```

**3단계: 구문 확인**

실행: `node -c lib/skills-core.js`
예상 결과: 출력 없음

**4단계: 커밋**

```bash
git add lib/skills-core.js
git commit -m "feat: add skill path resolution with shadowing support"
```

---

### 작업 4: 업데이트 확인 로직 추출

**파일:**
- 수정: `lib/skills-core.js`
- 참조: `.codex/superpowers-codex` (16~38행)

**1단계: checkForUpdates 함수 추가**

파일 상단의 require 문 뒤에 다음을 추가합니다:

```javascript
const { execSync } = require('child_process');
```

`module.exports` 앞에 다음을 추가합니다:

```javascript
/**
 * Check if a git repository has updates available.
 *
 * @param {string} repoDir - Path to git repository
 * @returns {boolean} - True if updates are available
 */
function checkForUpdates(repoDir) {
    try {
        // Quick check with 3 second timeout to avoid delays if network is down
        const output = execSync('git fetch origin && git status --porcelain=v1 --branch', {
            cwd: repoDir,
            timeout: 3000,
            encoding: 'utf8',
            stdio: 'pipe'
        });

        // Parse git status output to see if we're behind
        const statusLines = output.split('\n');
        for (const line of statusLines) {
            if (line.startsWith('## ') && line.includes('[behind ')) {
                return true; // We're behind remote
            }
        }
        return false; // Up to date
    } catch (error) {
        // Network down, git error, timeout, etc. - don't block bootstrap
        return false;
    }
}
```

**2단계: module.exports 업데이트**

```javascript
module.exports = {
    extractFrontmatter,
    findSkillsInDir,
    resolveSkillPath,
    checkForUpdates
};
```

**3단계: 구문 확인**

실행: `node -c lib/skills-core.js`
예상 결과: 출력 없음

**4단계: 커밋**

```bash
git add lib/skills-core.js
git commit -m "feat: add git update checking to core module"
```

---

## 2단계: 공유 코어를 사용하도록 Codex 리팩토링

### 작업 5: 공유 코어를 임포트하도록 Codex 업데이트

**파일:**
- 수정: `.codex/superpowers-codex` (상단에 임포트 추가)

**1단계: 임포트 구문 추가**

파일 상단의 기존 require 문 뒤(약 6행 부근)에 다음을 추가합니다:

```javascript
const skillsCore = require('../lib/skills-core');
```

**2단계: 구문 확인**

실행: `node -c .codex/superpowers-codex`
예상 결과: 출력 없음

**3단계: 커밋**

```bash
git add .codex/superpowers-codex
git commit -m "refactor: import shared skills core in codex"
```

---

### 작업 6: extractFrontmatter를 코어 버전으로 교체

**파일:**
- 수정: `.codex/superpowers-codex` (40~74행)

**1단계: 로컬 extractFrontmatter 함수 제거**

40~74행(extractFrontmatter 함수 정의 전체)을 삭제합니다.

**2단계: 모든 extractFrontmatter 호출 업데이트**

모든 `extractFrontmatter(` 호출을 찾아 `skillsCore.extractFrontmatter(`로 바꿉니다.

영향을 받는 행(대략): 90, 310

**3단계: 스크립트가 여전히 작동하는지 확인**

실행: `.codex/superpowers-codex find-skills | head -20`
예상 결과: 스킬 목록이 표시됨

**4단계: 커밋**

```bash
git add .codex/superpowers-codex
git commit -m "refactor: use shared extractFrontmatter in codex"
```

---

### 작업 7: findSkillsInDir를 코어 버전으로 교체

**파일:**
- 수정: `.codex/superpowers-codex` (약 97~136행)

**1단계: 로컬 findSkillsInDir 함수 제거**

`findSkillsInDir` 함수 정의 전체(약 97~136행)를 삭제합니다.

**2단계: 모든 findSkillsInDir 호출 업데이트**

`findSkillsInDir(` 호출을 `skillsCore.findSkillsInDir(`로 바꿉니다.

**3단계: 스크립트가 여전히 작동하는지 확인**

실행: `.codex/superpowers-codex find-skills | head -20`
예상 결과: 스킬 목록이 표시됨

**4단계: 커밋**

```bash
git add .codex/superpowers-codex
git commit -m "refactor: use shared findSkillsInDir in codex"
```

---

### 작업 8: checkForUpdates를 코어 버전으로 교체

**파일:**
- 수정: `.codex/superpowers-codex` (약 16~38행)

**1단계: 로컬 checkForUpdates 함수 제거**

`checkForUpdates` 함수 정의 전체를 삭제합니다.

**2단계: 모든 checkForUpdates 호출 업데이트**

`checkForUpdates(` 호출을 `skillsCore.checkForUpdates(`로 바꿉니다.

**3단계: 스크립트가 여전히 작동하는지 확인**

실행: `.codex/superpowers-codex bootstrap | head -50`
예상 결과: 부트스트랩 내용이 표시됨

**4단계: 커밋**

```bash
git add .codex/superpowers-codex
git commit -m "refactor: use shared checkForUpdates in codex"
```

---

## 3단계: OpenCode 플러그인 구축

### 작업 9: OpenCode 플러그인 디렉토리 구조 생성

**파일:**
- 생성: `.opencode/plugin/superpowers.js`

**1단계: 디렉토리 생성**

실행: `mkdir -p .opencode/plugin`

**2단계: 기본 플러그인 파일 생성**

```javascript
#!/usr/bin/env node

/**
 * Superpowers plugin for OpenCode.ai
 *
 * Provides custom tools for loading and discovering skills,
 * with automatic bootstrap on session start.
 */

const skillsCore = require('../../lib/skills-core');
const path = require('path');
const fs = require('fs');
const os = require('os');

const homeDir = os.homedir();
const superpowersSkillsDir = path.join(homeDir, '.config/opencode/superpowers/skills');
const personalSkillsDir = path.join(homeDir, '.config/opencode/skills');

/**
 * OpenCode plugin entry point
 */
export const SuperpowersPlugin = async ({ project, client, $, directory, worktree }) => {
  return {
    // Custom tools and hooks will go here
  };
};
```

**3단계: 파일 생성 확인**

실행: `ls -l .opencode/plugin/superpowers.js`
예상 결과: 파일이 존재함

**4단계: 커밋**

```bash
git add .opencode/plugin/superpowers.js
git commit -m "feat: create opencode plugin scaffold"
```

---

### 작업 10: use_skill 도구 구현

**파일:**
- 수정: `.opencode/plugin/superpowers.js`

**1단계: use_skill 도구 구현 추가**

플러그인 return 구문을 다음으로 교체합니다:

```javascript
export const SuperpowersPlugin = async ({ project, client, $, directory, worktree }) => {
  // Import zod for schema validation
  const { z } = await import('zod');

  return {
    tools: [
      {
        name: 'use_skill',
        description: 'Load and read a specific skill to guide your work. Skills contain proven workflows, mandatory processes, and expert techniques.',
        schema: z.object({
          skill_name: z.string().describe('Name of the skill to load (e.g., "superpowers:brainstorming" or "my-custom-skill")')
        }),
        execute: async ({ skill_name }) => {
          // Resolve skill path (handles shadowing: personal > superpowers)
          const resolved = skillsCore.resolveSkillPath(
            skill_name,
            superpowersSkillsDir,
            personalSkillsDir
          );

          if (!resolved) {
            return `Error: Skill "${skill_name}" not found.\n\nRun find_skills to see available skills.`;
          }

          // Read skill content
          const fullContent = fs.readFileSync(resolved.skillFile, 'utf8');
          const { name, description } = skillsCore.extractFrontmatter(resolved.skillFile);

          // Extract content after frontmatter
          const lines = fullContent.split('\n');
          let inFrontmatter = false;
          let frontmatterEnded = false;
          const contentLines = [];

          for (const line of lines) {
            if (line.trim() === '---') {
              if (inFrontmatter) {
                frontmatterEnded = true;
                continue;
              }
              inFrontmatter = true;
              continue;
            }

            if (frontmatterEnded || !inFrontmatter) {
              contentLines.push(line);
            }
          }

          const content = contentLines.join('\n').trim();
          const skillDirectory = path.dirname(resolved.skillFile);

          // Format output similar to Claude Code's Skill tool
          return `# ${name || skill_name}
# ${description || ''}
# Supporting tools and docs are in ${skillDirectory}
# ============================================

${content}`;
        }
      }
    ]
  };
};
```

**2단계: 구문 확인**

실행: `node -c .opencode/plugin/superpowers.js`
예상 결과: 출력 없음

**3단계: 커밋**

```bash
git add .opencode/plugin/superpowers.js
git commit -m "feat: implement use_skill tool for opencode"
```

---

### 작업 11: find_skills 도구 구현

**파일:**
- 수정: `.opencode/plugin/superpowers.js`

**1단계: tools 배열에 find_skills 도구 추가**

use_skill 도구 정의 뒤, tools 배열을 닫기 전에 다음을 추가합니다:

```javascript
      {
        name: 'find_skills',
        description: 'List all available skills in the superpowers and personal skill libraries.',
        schema: z.object({}),
        execute: async () => {
          // Find skills in both directories
          const superpowersSkills = skillsCore.findSkillsInDir(
            superpowersSkillsDir,
            'superpowers',
            3
          );
          const personalSkills = skillsCore.findSkillsInDir(
            personalSkillsDir,
            'personal',
            3
          );

          // Combine and format skills list
          const allSkills = [...personalSkills, ...superpowersSkills];

          if (allSkills.length === 0) {
            return 'No skills found. Install superpowers skills to ~/.config/opencode/superpowers/skills/';
          }

          let output = 'Available skills:\n\n';

          for (const skill of allSkills) {
            const namespace = skill.sourceType === 'personal' ? '' : 'superpowers:';
            const skillName = skill.name || path.basename(skill.path);

            output += `${namespace}${skillName}\n`;
            if (skill.description) {
              output += `  ${skill.description}\n`;
            }
            output += `  Directory: ${skill.path}\n\n`;
          }

          return output;
        }
      }
```

**2단계: 구문 확인**

실행: `node -c .opencode/plugin/superpowers.js`
예상 결과: 출력 없음

**3단계: 커밋**

```bash
git add .opencode/plugin/superpowers.js
git commit -m "feat: implement find_skills tool for opencode"
```

---

### 작업 12: 세션 시작 훅(Session Start Hook) 구현

**파일:**
- 수정: `.opencode/plugin/superpowers.js`

**1단계: session.started 훅 추가**

tools 배열 뒤에 다음을 추가합니다:

```javascript
    'session.started': async () => {
      // Read using-superpowers skill content
      const usingSuperpowersPath = skillsCore.resolveSkillPath(
        'using-superpowers',
        superpowersSkillsDir,
        personalSkillsDir
      );

      let usingSuperpowersContent = '';
      if (usingSuperpowersPath) {
        const fullContent = fs.readFileSync(usingSuperpowersPath.skillFile, 'utf8');
        // Strip frontmatter
        const lines = fullContent.split('\n');
        let inFrontmatter = false;
        let frontmatterEnded = false;
        const contentLines = [];

        for (const line of lines) {
          if (line.trim() === '---') {
            if (inFrontmatter) {
              frontmatterEnded = true;
              continue;
            }
            inFrontmatter = true;
            continue;
          }

          if (frontmatterEnded || !inFrontmatter) {
            contentLines.push(line);
          }
        }

        usingSuperpowersContent = contentLines.join('\n').trim();
      }

      // Tool mapping instructions
      const toolMapping = `
**Tool Mapping for OpenCode:**
When skills reference tools you don't have, substitute OpenCode equivalents:
- \`TodoWrite\` → \`update_plan\` (your planning/task tracking tool)
- \`Task\` tool with subagents → Use OpenCode's subagent system (@mention syntax or automatic dispatch)
- \`Skill\` tool → \`use_skill\` custom tool (already available)
- \`Read\`, \`Write\`, \`Edit\`, \`Bash\` → Use your native tools

**Skill directories contain supporting files:**
- Scripts you can run with bash tool
- Additional documentation you can read
- Utilities and helpers specific to that skill

**Skills naming:**
- Superpowers skills: \`superpowers:skill-name\` (from ~/.config/opencode/superpowers/skills/)
- Personal skills: \`skill-name\` (from ~/.config/opencode/skills/)
- Personal skills override superpowers skills when names match
`;

      // Check for updates (non-blocking)
      const hasUpdates = skillsCore.checkForUpdates(
        path.join(homeDir, '.config/opencode/superpowers')
      );

      const updateNotice = hasUpdates ?
        '\n\n⚠️ **Updates available!** Run `cd ~/.config/opencode/superpowers && git pull` to update superpowers.' :
        '';

      // Return context to inject into session
      return {
        context: `<EXTREMELY_IMPORTANT>
You have superpowers.

**Below is the full content of your 'superpowers:using-superpowers' skill - your introduction to using skills. For all other skills, use the 'use_skill' tool:**

${usingSuperpowersContent}

${toolMapping}${updateNotice}
</EXTREMELY_IMPORTANT>`
      };
    }
```

**2단계: 구문 확인**

실행: `node -c .opencode/plugin/superpowers.js`
예상 결과: 출력 없음

**3단계: 커밋**

```bash
git add .opencode/plugin/superpowers.js
git commit -m "feat: implement session.started hook for opencode"
```

---

## 4단계: 문서화

### 작업 13: OpenCode 설치 가이드 생성

**파일:**
- 생성: `.opencode/INSTALL.md`

**1단계: 설치 가이드 생성**

````markdown
# Installing Superpowers for OpenCode

## Prerequisites

- [OpenCode.ai](https://opencode.ai) installed
- Node.js installed
- Git installed

## Installation Steps

### 1. Install Superpowers Skills

```bash
# Clone superpowers skills to OpenCode config directory
mkdir -p ~/.config/opencode/superpowers
git clone https://github.com/obra/superpowers.git ~/.config/opencode/superpowers
```

### 2. Install the Plugin

The plugin is included in the superpowers repository you just cloned.

OpenCode will automatically discover it from:
- `~/.config/opencode/superpowers/.opencode/plugin/superpowers.js`

Or you can link it to the project-local plugin directory:

```bash
# In your OpenCode project
mkdir -p .opencode/plugin
ln -s ~/.config/opencode/superpowers/.opencode/plugin/superpowers.js .opencode/plugin/superpowers.js
```

### 3. Restart OpenCode

Restart OpenCode to load the plugin. On the next session, you should see:

```
You have superpowers.
```

## Usage

### Finding Skills

Use the `find_skills` tool to list all available skills:

```
use find_skills tool
```

### Loading a Skill

Use the `use_skill` tool to load a specific skill:

```
use use_skill tool with skill_name: "superpowers:brainstorming"
```

### Personal Skills

Create your own skills in `~/.config/opencode/skills/`:

```bash
mkdir -p ~/.config/opencode/skills/my-skill
```

Create `~/.config/opencode/skills/my-skill/SKILL.md`:

```markdown
---
name: my-skill
description: Use when [condition] - [what it does]
---

# My Skill

[Your skill content here]
```

Personal skills override superpowers skills with the same name.

## Updating

```bash
cd ~/.config/opencode/superpowers
git pull
```

## Troubleshooting

### Plugin not loading

1. Check plugin file exists: `ls ~/.config/opencode/superpowers/.opencode/plugin/superpowers.js`
2. Check OpenCode logs for errors
3. Verify Node.js is installed: `node --version`

### Skills not found

1. Verify skills directory exists: `ls ~/.config/opencode/superpowers/skills`
2. Use `find_skills` tool to see what's discovered
3. Check file structure: each skill should have a `SKILL.md` file

### Tool mapping issues

When a skill references a Claude Code tool you don't have:
- `TodoWrite` → use `update_plan`
- `Task` with subagents → use `@mention` syntax to invoke OpenCode subagents
- `Skill` → use `use_skill` tool
- File operations → use your native tools

## Getting Help

- Report issues: https://github.com/obra/superpowers/issues
- Documentation: https://github.com/obra/superpowers
````

**2단계: 파일 생성 확인**

실행: `ls -l .opencode/INSTALL.md`
예상 결과: 파일이 존재함

**3단계: 커밋**

```bash
git add .opencode/INSTALL.md
git commit -m "docs: add opencode installation guide"
```

---

### 작업 14: 메인 README 업데이트

**파일:**
- 수정: `README.md`

**1단계: OpenCode 섹션 추가**

지원되는 플랫폼에 대한 섹션을 찾아(파일에서 "Codex" 검색) 그 뒤에 다음을 추가합니다:

```markdown
### OpenCode

Superpowers works with [OpenCode.ai](https://opencode.ai) through a native JavaScript plugin.

**Installation:** See [.opencode/INSTALL.md](.opencode/INSTALL.md)

**Features:**
- Custom tools: `use_skill` and `find_skills`
- Automatic session bootstrap
- Personal skills with shadowing
- Supporting files and scripts access
```

**2단계: 포맷 확인**

실행: `grep -A 10 "### OpenCode" README.md`
예상 결과: 추가한 섹션이 표시됨

**3단계: 커밋**

```bash
git add README.md
git commit -m "docs: add opencode support to readme"
```

---

### 작업 15: 릴리스 노트 업데이트

**파일:**
- 수정: `RELEASE-NOTES.md`

**1단계: OpenCode 지원 항목 추가**

파일 상단(헤더 바로 뒤)에 다음을 추가합니다:

```markdown
## [Unreleased]

### Added

- **OpenCode Support**: Native JavaScript plugin for OpenCode.ai
  - Custom tools: `use_skill` and `find_skills`
  - Automatic session bootstrap with tool mapping instructions
  - Shared core module (`lib/skills-core.js`) for code reuse
  - Installation guide in `.opencode/INSTALL.md`

### Changed

- **Refactored Codex Implementation**: Now uses shared `lib/skills-core.js` module
  - Eliminates code duplication between Codex and OpenCode
  - Single source of truth for skill discovery and parsing

---

```

**2단계: 포맷 확인**

실행: `head -30 RELEASE-NOTES.md`
예상 결과: 새로 추가한 섹션이 표시됨

**3단계: 커밋**

```bash
git add RELEASE-NOTES.md
git commit -m "docs: add opencode support to release notes"
```

---

## 5단계: 최종 검증

### 작업 16: Codex가 여전히 작동하는지 테스트

**파일:**
- 테스트: `.codex/superpowers-codex`

**1단계: find-skills 명령어 테스트**

실행: `.codex/superpowers-codex find-skills | head -20`
예상 결과: 이름과 설명이 포함된 스킬 목록이 표시됨

**2단계: use-skill 명령어 테스트**

실행: `.codex/superpowers-codex use-skill superpowers:brainstorming | head -20`
예상 결과: 브레인스토밍 스킬 내용이 표시됨

**3단계: bootstrap 명령어 테스트**

실행: `.codex/superpowers-codex bootstrap | head -30`
예상 결과: 지침이 포함된 부트스트랩 내용이 표시됨

**4단계: 모든 테스트 통과 시 성공 기록**

커밋 불필요 - 검증 전용 단계입니다.

---

### 작업 17: 파일 구조 확인

**파일:**
- 확인: 모든 신규 파일 존재 여부

**1단계: 생성된 모든 파일 확인**

실행:
```bash
ls -l lib/skills-core.js
ls -l .opencode/plugin/superpowers.js
ls -l .opencode/INSTALL.md
```

예상 결과: 모든 파일이 존재함

**2단계: 디렉토리 구조 확인**

실행: `tree -L 2 .opencode/` (또는 tree를 사용할 수 없는 경우 `find .opencode -type f`)
예상 결과:
```
.opencode/
├── INSTALL.md
└── plugin/
    └── superpowers.js
```

**3단계: 구조가 올바르면 다음으로 진행**

커밋 불필요 - 검증 전용 단계입니다.

---

### 작업 18: 최종 커밋 및 요약

**파일:**
- 확인: `git status`

**1단계: git 상태 확인**

실행: `git status`
예상 결과: 워킹 트리가 깨끗하며 모든 변경 사항이 커밋됨

**2단계: 커밋 로그 검토**

실행: `git log --oneline -20`
예상 결과: 이번 구현의 모든 커밋이 표시됨

**3단계: 요약 문서 작성**

다음 내용을 보여주는 완료 요약을 작성합니다:
- 수행한 총 커밋 수
- 생성된 파일: `lib/skills-core.js`, `.opencode/plugin/superpowers.js`, `.opencode/INSTALL.md`
- 수정된 파일: `.codex/superpowers-codex`, `README.md`, `RELEASE-NOTES.md`
- 수행된 테스트: Codex 명령어 검증 완료
- 준비 완료 항목: 실제 OpenCode 설치 환경에서의 테스트

**4단계: 완료 보고**

사용자에게 요약을 제시하고 다음 작업을 제안합니다:
1. 원격 저장소에 푸시(Push)
2. 풀 리퀘스트(Pull Request) 생성
3. 실제 OpenCode 설치 환경에서 테스트 (OpenCode 설치 필요)

---

## 테스트 가이드 (수동 - OpenCode 필요)

이 단계들은 OpenCode가 설치되어 있어야 하며 자동화된 구현에는 포함되지 않습니다:

1. **스킬 설치**: `.opencode/INSTALL.md`의 지침을 따릅니다.
2. **OpenCode 세션 시작**: 부트스트랩이 나타나는지 확인합니다.
3. **find_skills 테스트**: 사용 가능한 모든 스킬이 나열되어야 합니다.
4. **use_skill 테스트**: 스킬을 로드하고 내용이 표시되는지 확인합니다.
5. **보조 파일 테스트**: 스킬 디렉토리 경로에 접근할 수 있는지 확인합니다.
6. **개인 스킬 테스트**: 개인 스킬을 생성하고 코어 스킬을 섀도잉(대체)하는지 확인합니다.
7. **도구 매핑 테스트**: TodoWrite → update_plan 매핑이 작동하는지 확인합니다.

## 성공 기준

- [ ] 모든 핵심 함수가 포함된 `lib/skills-core.js` 생성됨
- [ ] 공유 코어를 사용하도록 `.codex/superpowers-codex` 리팩토링됨
- [ ] Codex 명령어들(find-skills, use-skill, bootstrap)이 여전히 작동함
- [ ] 도구와 훅이 포함된 `.opencode/plugin/superpowers.js` 생성됨
- [ ] 설치 가이드 생성됨
- [ ] README 및 RELEASE-NOTES 업데이트됨
- [ ] 모든 변경 사항이 커밋됨
- [ ] 워킹 트리가 깨끗함
