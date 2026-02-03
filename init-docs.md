# Initialize Agent Documentation

You are setting up the `.agents/` documentation structure for just-in-time AI agent context.

**⚠️ IMPORTANT**: This skill is synchronized with `audit-docs.md`. Any changes to structure, templates, or standards here MUST be reflected in `audit-docs.md`. Review both files when making updates.

## Phase 1: Detect Project Type

Scan the repository for indicators:

| Indicator | Project Type |
|-----------|--------------|
| `package.json` with React/Vue/Angular | frontend |
| `package.json` with Express/Fastify/Nest | backend/api |
| `.csproj` with ASP.NET | backend/api |
| `Dockerfile` + queue/worker keywords | jobs/workers |
| SQL migrations folder, `.sql` files | database |
| `schema.prisma`, `migrations/` | database |

If unclear, default to `backend/api` as the most common.

## Phase 2: Interview User

Use `AskUserQuestion` to confirm:

1. **Project type**: "Detected [type]. Is this correct?"
   - Options: frontend, backend/api, database, jobs/workers, Other

2. **Product name**: "What is the product name? (Used in product-overview.md)"
   - Free text response expected

3. **Project role**: "In one sentence, what does THIS repo do within the product?"
   - Free text response expected (this becomes project-overview.md)

## Phase 3: Create Structure

Create the following structure. **All folders must be created even if empty.**

```
.agents/
├── AI-INSTRUCTIONS.md
├── product-overview.md
├── project-overview.md
├── operations/
│   └── INDEX.md
├── reference/
│   └── INDEX.md
├── architecture/
│   ├── INDEX.md
│   └── integrations/
│       └── INDEX.md
├── scripts/
│   ├── lint-docs.js
│   └── lint-structure.js
└── .scratch/
    └── .gitkeep
```

### File Contents

**AI-INSTRUCTIONS.md** (use template below, fill in [PRODUCT_NAME]):
```markdown
# Instructions for AI Agents

This documentation helps you understand [PRODUCT_NAME]. Read this file first.

## How to Find What You Need

1. **Start here**: Read `product-overview.md` for product context
2. **This repo**: Read `project-overview.md` for this repo's role
3. **Find operations**: Search `operations/INDEX.md` for domain operations
4. **Find terminology**: Search `reference/INDEX.md` for definitions
5. **Find patterns**: Search `architecture/INDEX.md` for cross-cutting concerns
6. **Load on-demand**: Only load docs as needed for your task

## Documentation Structure

.agents/
├── AI-INSTRUCTIONS.md       # Start here (this file)
├── product-overview.md      # What is the product?
├── project-overview.md      # What does this repo do?
├── operations/              # Domain operations (grouped by domain)
│   └── INDEX.md
├── reference/               # Terminology and definitions
│   └── INDEX.md
├── architecture/            # Patterns and integrations
│   ├── INDEX.md
│   └── integrations/        # External service dependencies
│       └── INDEX.md
└── scripts/                 # Utility scripts

## Token Limits

Run `npm run lint:docs` to check limits before committing.

| Doc Type | Limit | Purpose |
|----------|-------|---------|
| AI-INSTRUCTIONS.md | 1000 | Entry point |
| product-overview.md | 500 | Product context |
| project-overview.md | 100 | Repo role (brief!) |
| operations/*.md | 500 | Domain operations |
| reference/*.md | 500 | Terminology |
| architecture/*.md | 500 | Patterns |
| **/INDEX.md | 200 | Keyword indexes |

## Templates

### Operations (operations/[domain].md)
Group related operations by domain. Example: users.md covers create, update, delete.

### Reference (reference/[concept].md)
Definitions and lookup tables. No how-to steps.

### Architecture (architecture/[pattern].md)
Cross-cutting patterns: auth, caching, error handling, etc.

### Integrations (architecture/integrations/[service].md)
External services this repo depends on: APIs, databases, queues.

## Writing Rules

1. **Operations = what you can do**: Domain actions, API endpoints, jobs
2. **Reference = definitions**: Reusable lookups, no action steps
3. **Architecture = how things work**: Patterns, not step-by-step
4. **Brief is better**: Stay under token limits, link for details
```

**product-overview.md**:
```markdown
# [PRODUCT_NAME] Product Overview

[Describe what the product is and who uses it]

## Who Uses It

- **[Role 1]** - [What they do]
- **[Role 2]** - [What they do]

## Core Capabilities

| Capability | Description |
|------------|-------------|
| [Capability 1] | [Description] |

## Key Terminology

| Term | Meaning |
|------|---------|
| [Term] | [Definition] |
```

**project-overview.md**:
```markdown
# Project Overview

[PROJECT_ROLE - one sentence describing this repo's role]

**Type**: [PROJECT_TYPE]
**Repo**: [Repo name or path]
```

**INDEX.md files** (all use same format):
```markdown
# [Folder Name] Index

Search this file for keywords. Add entries as documentation grows.

---

<!-- Example entry format:
**Keywords**: keyword1, keyword2, keyword3
- [Document Name](./document.md)
-->
```

**lint-docs.js** (create exactly as shown):
```javascript
#!/usr/bin/env node

const fs = require('fs');
const path = require('path');

const colors = {
  red: '\x1b[31m',
  yellow: '\x1b[33m',
  green: '\x1b[32m',
  reset: '\x1b[0m'
};

function getColor(percent) {
  if (percent > 100) return colors.red;
  if (percent > 90) return colors.yellow;
  return colors.green;
}

const LIMITS = {
  'AI-INSTRUCTIONS.md': 1000,
  'product-overview.md': 500,
  'project-overview.md': 100,
  'INDEX.md': 200,
  'operations': 500,
  'workflows': 500,
  'reference': 500,
  'architecture': 500,
  'integrations': 500,
  'default': 500
};

function estimateTokens(text) {
  const words = text.split(/\s+/).filter(w => w.length > 0).length;
  const codeBlocks = (text.match(/```[\s\S]*?```/g) || []).length;
  const specialChars = (text.match(/[|#*\->`]/g) || []).length;
  return Math.ceil(words * 1.3 + specialChars * 0.1 + codeBlocks * 10);
}

let tokenize = null;
try {
  const { encode } = require('gpt-tokenizer');
  tokenize = (text) => encode(text).length;
  console.log('Using gpt-tokenizer for accurate token counts\n');
} catch {
  tokenize = estimateTokens;
  console.log('Using word-based estimation (install gpt-tokenizer for accuracy)\n');
}

function getLimit(filePath) {
  const fileName = path.basename(filePath);
  const parentDir = path.basename(path.dirname(filePath));
  if (fileName === 'AI-INSTRUCTIONS.md') return LIMITS['AI-INSTRUCTIONS.md'];
  if (fileName === 'product-overview.md') return LIMITS['product-overview.md'];
  if (fileName === 'project-overview.md') return LIMITS['project-overview.md'];
  if (fileName === 'INDEX.md') return LIMITS['INDEX.md'];
  if (parentDir === 'operations') return LIMITS['operations'];
  if (parentDir === 'workflows') return LIMITS['workflows'];
  if (parentDir === 'reference') return LIMITS['reference'];
  if (parentDir === 'architecture') return LIMITS['architecture'];
  if (parentDir === 'integrations') return LIMITS['integrations'];
  return LIMITS['default'];
}

function getLimitLabel(filePath) {
  const fileName = path.basename(filePath);
  const parentDir = path.basename(path.dirname(filePath));
  if (fileName === 'AI-INSTRUCTIONS.md') return 'base doc';
  if (fileName === 'product-overview.md') return 'product';
  if (fileName === 'project-overview.md') return 'project';
  if (fileName === 'INDEX.md') return 'index';
  if (parentDir === 'operations') return 'operation';
  if (parentDir === 'workflows') return 'workflow';
  if (parentDir === 'reference') return 'reference';
  if (parentDir === 'architecture') return 'architecture';
  if (parentDir === 'integrations') return 'integration';
  return 'doc';
}

function findMarkdownFiles(dir) {
  const files = [];
  const skipDirs = ['scripts', '.scratch'];
  function walk(currentDir) {
    if (!fs.existsSync(currentDir)) return;
    const entries = fs.readdirSync(currentDir, { withFileTypes: true });
    for (const entry of entries) {
      const fullPath = path.join(currentDir, entry.name);
      if (entry.isDirectory() && !skipDirs.includes(entry.name)) {
        walk(fullPath);
      } else if (entry.isFile() && entry.name.endsWith('.md')) {
        files.push(fullPath);
      }
    }
  }
  walk(dir);
  return files;
}

function createBar(percent) {
  const width = 20;
  const filled = Math.min(Math.round((percent / 100) * width), width);
  const empty = width - filled;
  const overFilled = percent > 100 ? Math.min(percent - 100, 50) / 50 * 5 : 0;
  const color = getColor(percent);
  let bar = '█'.repeat(filled) + '░'.repeat(empty);
  if (percent > 100) {
    bar = '█'.repeat(width) + '▓'.repeat(Math.round(overFilled));
  }
  return `${color}[${bar}] ${percent}%${colors.reset}`;
}

function lintDocs() {
  const agentsDir = path.resolve(__dirname, '..');
  const files = findMarkdownFiles(agentsDir);
  if (files.length === 0) {
    console.log('No markdown files found in .agents/');
    return 0;
  }
  const results = [];
  let hasErrors = false;
  for (const filePath of files) {
    const content = fs.readFileSync(filePath, 'utf-8');
    const tokens = tokenize(content);
    const limit = getLimit(filePath);
    const relativePath = path.relative(agentsDir, filePath);
    const label = getLimitLabel(filePath);
    const status = tokens <= limit ? 'PASS' : 'FAIL';
    const percent = Math.round((tokens / limit) * 100);
    if (tokens > limit) hasErrors = true;
    results.push({ path: relativePath, tokens, limit, label, status, percent, over: tokens - limit });
  }
  results.sort((a, b) => {
    if (a.status !== b.status) return a.status === 'FAIL' ? -1 : 1;
    return b.tokens - a.tokens;
  });
  console.log('Limits: base=1000 | product=500 | project=100 | ops/ref/arch=500 | index=200\n');
  console.log('Token Limits for .agents/ Documentation');
  console.log('═'.repeat(60));
  for (const r of results) {
    const color = getColor(r.percent);
    const icon = r.status === 'PASS' ? '✓' : '✗';
    const bar = createBar(r.percent);
    console.log(`${color}${icon} ${r.path}${colors.reset}`);
    console.log(`  ${bar} ${r.tokens}/${r.limit} tokens (${r.label})`);
    if (r.status === 'FAIL') {
      console.log(`  ${colors.red}⚠ Over by ${r.over} tokens - needs trimming${colors.reset}`);
    }
    console.log();
  }
  console.log('═'.repeat(60));
  const passed = results.filter(r => r.status === 'PASS').length;
  const failed = results.filter(r => r.status === 'FAIL').length;
  const totalTokens = results.reduce((sum, r) => sum + r.tokens, 0);
  console.log(`Total: ${totalTokens} tokens across ${results.length} files`);
  console.log(`${colors.green}Passed: ${passed}${colors.reset} | ${failed > 0 ? colors.red : colors.green}Failed: ${failed}${colors.reset}`);
  if (hasErrors) {
    console.log(`\n${colors.red}✗ Documentation exceeds token limits${colors.reset}`);
    return 1;
  } else {
    console.log(`\n${colors.green}✓ All documentation within limits${colors.reset}`);
    return 0;
  }
}

const exitCode = lintDocs();
process.exit(exitCode);
```

**lint-structure.js** (create exactly as shown):
```javascript
#!/usr/bin/env node

const fs = require('fs');
const path = require('path');

const colors = {
  red: '\x1b[31m',
  yellow: '\x1b[33m',
  green: '\x1b[32m',
  cyan: '\x1b[36m',
  reset: '\x1b[0m'
};

const REQUIRED_FILES = ['AI-INSTRUCTIONS.md', 'product-overview.md', 'project-overview.md'];
const REQUIRED_FOLDERS = ['operations', 'reference', 'architecture', 'scripts'];
const OPTIONAL_ITEMS = ['.scratch'];
const ARCHITECTURE_REQUIRED = ['integrations'];

function lintStructure() {
  const agentsDir = path.resolve(__dirname, '..');
  const errors = [];
  const passes = [];
  console.log(`${colors.cyan}Structure Linter for .agents/${colors.reset}\n`);
  if (!fs.existsSync(agentsDir)) {
    console.log(`${colors.red}✗ .agents/ directory does not exist${colors.reset}`);
    console.log(`  Run /init-docs to create the structure\n`);
    return 1;
  }
  const rootItems = fs.readdirSync(agentsDir, { withFileTypes: true });
  const rootFiles = rootItems.filter(i => i.isFile()).map(i => i.name);
  const rootFolders = rootItems.filter(i => i.isDirectory()).map(i => i.name);
  console.log('Required Files:');
  for (const file of REQUIRED_FILES) {
    const filePath = path.join(agentsDir, file);
    if (fs.existsSync(filePath)) {
      passes.push(`Required file: ${file}`);
      console.log(`  ${colors.green}✓${colors.reset} ${file}`);
    } else {
      errors.push(`Missing required file: ${file}`);
      console.log(`  ${colors.red}✗${colors.reset} ${file} ${colors.red}(missing)${colors.reset}`);
    }
  }
  console.log('\nRequired Folders:');
  for (const folder of REQUIRED_FOLDERS) {
    const folderPath = path.join(agentsDir, folder);
    if (fs.existsSync(folderPath) && fs.statSync(folderPath).isDirectory()) {
      passes.push(`Required folder: ${folder}/`);
      console.log(`  ${colors.green}✓${colors.reset} ${folder}/`);
    } else {
      errors.push(`Missing required folder: ${folder}/`);
      console.log(`  ${colors.red}✗${colors.reset} ${folder}/ ${colors.red}(missing)${colors.reset}`);
    }
  }
  const allowedItems = [...REQUIRED_FILES, ...REQUIRED_FOLDERS, ...OPTIONAL_ITEMS];
  const allRootItems = [...rootFiles, ...rootFolders];
  const unexpectedItems = allRootItems.filter(item => !allowedItems.includes(item));
  if (unexpectedItems.length > 0) {
    console.log('\nUnexpected Items at Root:');
    for (const item of unexpectedItems) {
      const isDir = rootFolders.includes(item);
      errors.push(`Unexpected at root: ${item}${isDir ? '/' : ''}`);
      console.log(`  ${colors.red}✗${colors.reset} ${item}${isDir ? '/' : ''} ${colors.red}(not allowed)${colors.reset}`);
    }
  }
  const archPath = path.join(agentsDir, 'architecture');
  if (fs.existsSync(archPath)) {
    console.log('\nArchitecture Structure:');
    for (const required of ARCHITECTURE_REQUIRED) {
      const subPath = path.join(archPath, required);
      if (fs.existsSync(subPath) && fs.statSync(subPath).isDirectory()) {
        passes.push(`architecture/${required}/ exists`);
        console.log(`  ${colors.green}✓${colors.reset} architecture/${required}/`);
      } else {
        errors.push(`Missing: architecture/${required}/`);
        console.log(`  ${colors.red}✗${colors.reset} architecture/${required}/ ${colors.red}(missing)${colors.reset}`);
      }
    }
  }
  console.log('\nINDEX.md Checks:');
  const foldersToCheck = [...REQUIRED_FOLDERS.filter(f => f !== 'scripts'), 'architecture/integrations'];
  for (const folderRel of foldersToCheck) {
    const folderPath = path.join(agentsDir, folderRel);
    if (!fs.existsSync(folderPath)) continue;
    const items = fs.readdirSync(folderPath, { withFileTypes: true });
    const mdFiles = items.filter(i => i.isFile() && i.name.endsWith('.md'));
    const hasIndex = mdFiles.some(f => f.name === 'INDEX.md');
    const isMainFolder = REQUIRED_FOLDERS.includes(folderRel) || folderRel === 'architecture/integrations';
    if (isMainFolder || mdFiles.length > 0) {
      if (hasIndex) {
        passes.push(`${folderRel}/INDEX.md exists`);
        console.log(`  ${colors.green}✓${colors.reset} ${folderRel}/INDEX.md`);
      } else {
        errors.push(`Missing: ${folderRel}/INDEX.md`);
        console.log(`  ${colors.red}✗${colors.reset} ${folderRel}/INDEX.md ${colors.red}(missing)${colors.reset}`);
      }
    }
  }
  for (const folder of ['operations', 'reference', 'architecture']) {
    const folderPath = path.join(agentsDir, folder);
    if (!fs.existsSync(folderPath)) continue;
    checkSubfoldersForIndex(folderPath, folder, errors, passes);
  }
  console.log('\n' + '═'.repeat(50));
  console.log(`${colors.green}Passed: ${passes.length}${colors.reset}`);
  if (errors.length > 0) {
    console.log(`${colors.red}Failed: ${errors.length}${colors.reset}`);
    console.log(`\n${colors.red}✗ Structure validation failed${colors.reset}`);
    return 1;
  } else {
    console.log(`\n${colors.green}✓ Structure is valid${colors.reset}`);
    return 0;
  }
}

function checkSubfoldersForIndex(folderPath, relativePath, errors, passes) {
  const items = fs.readdirSync(folderPath, { withFileTypes: true });
  const subfolders = items.filter(i => i.isDirectory() && !i.name.startsWith('.'));
  for (const subfolder of subfolders) {
    if (relativePath === 'architecture' && subfolder.name === 'integrations') continue;
    const subPath = path.join(folderPath, subfolder.name);
    const subRel = `${relativePath}/${subfolder.name}`;
    const subItems = fs.readdirSync(subPath, { withFileTypes: true });
    const hasMdFiles = subItems.some(i => i.isFile() && i.name.endsWith('.md'));
    const hasIndex = subItems.some(i => i.isFile() && i.name === 'INDEX.md');
    if (hasMdFiles && !hasIndex) {
      errors.push(`Missing: ${subRel}/INDEX.md (folder has .md files)`);
      console.log(`  ${colors.red}✗${colors.reset} ${subRel}/INDEX.md ${colors.red}(folder has docs but no index)${colors.reset}`);
    }
    checkSubfoldersForIndex(subPath, subRel, errors, passes);
  }
}

const exitCode = lintStructure();
process.exit(exitCode);
```

## Phase 4: Update .gitignore

Append to `.gitignore` (create if doesn't exist):
```
# AI agent scratch space
.agents/.scratch/
```

## Phase 5: Add npm Scripts

If `package.json` exists, add these scripts:
```json
"lint:docs": "node .agents/scripts/lint-docs.js",
"lint:structure": "node .agents/scripts/lint-structure.js",
"lint:agents": "node .agents/scripts/lint-structure.js && node .agents/scripts/lint-docs.js"
```

## Phase 6: Report

When complete, output:

```
## Documentation Initialized

**Product**: [PRODUCT_NAME]
**Project Type**: [TYPE]
**Project Role**: [ONE_SENTENCE]

### Created Structure
- .agents/AI-INSTRUCTIONS.md
- .agents/product-overview.md
- .agents/project-overview.md
- .agents/operations/INDEX.md
- .agents/reference/INDEX.md
- .agents/architecture/INDEX.md
- .agents/architecture/integrations/INDEX.md
- .agents/scripts/lint-docs.js
- .agents/scripts/lint-structure.js
- .agents/.scratch/.gitkeep
- Updated .gitignore
- Added npm scripts (if package.json exists)

### Next Steps
1. Fill in product-overview.md with product details
2. Run `npm run lint:agents` to verify setup
3. Add operations/reference/architecture docs as needed
```

## Rules

- **DO NOT fill in operations/reference/architecture content** - only create empty INDEX.md stubs
- **Always create all folders** even if the project type doesn't obviously need them
- **Keep INDEX.md minimal** - just header and example comment
- **project-overview.md must be under 100 tokens** - enforce brevity
