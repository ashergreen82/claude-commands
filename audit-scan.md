# Audit Scan - Documentation Structure Analysis

You are scanning the `.agents/` directory to identify non-standard structure and migration candidates.

## Prerequisites

Check if `.agents/` exists. If not, inform user to run `/init-docs` first and stop.

## Execute Scan

Spawn an **Explore subagent** with this prompt:

```
Scan the .agents/ directory and report findings in JSON format.

Check for:

1. NON-STANDARD ROOT FILES - Files other than:
   - AI-INSTRUCTIONS.md
   - product-overview.md
   - project-overview.md

2. NON-STANDARD ROOT FOLDERS - Folders other than:
   - operations/
   - reference/
   - architecture/
   - scripts/
   - .scratch/

3. NESTED STRUCTURES - Look for:
   - docs/ subdirectory with nested folders
   - technical/, product/, infrastructure/, quickstart/ folders
   - Any multi-level folder hierarchies

4. MISSING STANDARD ITEMS - Check if required folders/files exist

5. MISSING LINT SCRIPTS - Check for:
   - .agents/scripts/lint-docs.js
   - .agents/scripts/lint-structure.js

6. GIT HOOKS STATUS - Check:
   - .git/hooks/pre-commit exists?
   - Does it reference lint-structure.js or lint-docs.js?
   - Or check .husky/pre-commit, package.json lint-staged config

Return findings as JSON:
{
  "nonStandardFiles": ["file1.md", "file2.md"],
  "nonStandardFolders": ["docs/", "old-docs/"],
  "nestedStructures": [
    {"path": "docs/infrastructure/", "files": ["NGINX.md", "PM2.md"]},
    {"path": "docs/quickstart/", "files": ["DEPLOYMENT.md"]}
  ],
  "missingStandard": {
    "files": ["product-overview.md"],
    "folders": ["reference/"]
  },
  "lintScripts": {
    "lintDocs": true|false,
    "lintStructure": true|false
  },
  "gitHooks": {
    "preCommitExists": true|false,
    "runsLinting": true|false,
    "hookType": "native|husky|lint-staged|none"
  },
  "totalMdFiles": 15,
  "estimatedMigrations": 8
}
```

## Save State

Write the findings to `.agents/.scratch/audit-state.json`:

```json
{
  "phase": "scanned",
  "timestamp": "ISO timestamp",
  "findings": { /* subagent response */ }
}
```

Create `.agents/.scratch/` directory if it doesn't exist.

## Report to User

Present a human-readable summary:

```markdown
## Audit Scan Complete

### Current Structure
- Total .md files: X
- Standard structure: [Complete|Partial|Missing]

### Non-Standard Items Found
| Type | Items |
|------|-------|
| Root files | file1.md, file2.md |
| Root folders | docs/, old-docs/ |
| Nested structures | docs/infrastructure/ (3 files), docs/quickstart/ (2 files) |

### Missing Standard Items
- [ ] product-overview.md
- [ ] reference/ folder

### Tooling Status
- Lint scripts: [Installed|Missing]
- Git hooks: [Configured|Missing|Partial]

### Next Step
Run `/audit-migrate` to plan and execute migrations.
```

## Rules

- **DO NOT modify any files** - this is read-only scanning
- **DO NOT interview user** - just report findings
- **ALWAYS save state** - enables `/audit-migrate` to continue
