# Audit Validate - Verify Documentation Structure

You are validating the `.agents/` documentation structure after migration.

## Phase 1: Check Prerequisites

Verify `.agents/` directory exists. If not, inform user to run `/init-docs` first.

Check for lint scripts:
- `.agents/scripts/lint-structure.js`
- `.agents/scripts/lint-docs.js`

If missing, offer to install them before validation.

## Phase 2: Run Validation

Spawn a **Bash subagent** to run lint scripts:

```
Run documentation validation for .agents/ directory.

Execute in sequence:
1. node .agents/scripts/lint-structure.js
2. node .agents/scripts/lint-docs.js

Capture full output from both commands.
Report exit codes.
```

## Phase 3: Parse Results

Analyze the lint output for:

**Structure issues:**
- Missing required files
- Missing required folders
- Unexpected items at root
- Missing INDEX.md files

**Token limit issues:**
- Files exceeding limits
- Which files and by how much

## Phase 4: Load Migration State (Optional)

If `.agents/.scratch/audit-state.json` exists with phase="migrated":
- Include migration summary in report
- Compare before/after state
- Clean up state file after successful validation

## Phase 5: Report

```markdown
## Validation Results

### Structure Check
[✓|✗] Required files present
[✓|✗] Required folders present
[✓|✗] No unexpected root items
[✓|✗] All INDEX.md files present

### Token Limits
| File | Tokens | Limit | Status |
|------|--------|-------|--------|
| AI-INSTRUCTIONS.md | 450 | 1000 | ✓ |
| product-overview.md | 520 | 500 | ✗ Over by 20 |

### Overall
- Structure: [PASS|FAIL]
- Token limits: [PASS|FAIL]
- Total tokens: XXXX across XX files

### Issues Found (if any)
1. product-overview.md exceeds limit by 20 tokens - consider trimming
2. reference/INDEX.md is missing

### Migration Summary (if applicable)
- Files migrated: X
- Folders cleaned: X
- State file: [Cleaned up|Retained for review]
```

## Phase 6: Suggest Fixes

If validation failed, provide actionable suggestions:

**For structure issues:**
```markdown
### Fix: Missing INDEX.md
Run: touch .agents/reference/INDEX.md
Then add keyword entries for existing docs.
```

**For token limit issues:**
```markdown
### Fix: product-overview.md over limit
Current: 520 tokens | Limit: 500 tokens

Suggestions:
1. Move detailed sections to reference/ docs
2. Remove redundant content
3. Use bullet points instead of paragraphs
```

## Phase 7: Cleanup

If validation passed and state file exists:

Ask user:
```
"Validation passed. Clean up audit state file?"
Options:
- Yes, remove .agents/.scratch/audit-state.json
- Keep for reference
```

## Rules

- **Read-only** except for optional state cleanup
- **Always run both lint scripts** for complete validation
- **Provide actionable fixes** for any failures
