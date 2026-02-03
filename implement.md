# Implementation Mode

Documentation has been reviewed. Now implement the fix.

## Phase 1: Load Context

Read relevant documentation before coding:

1. **Operations**: Read `.agents/operations/` docs for the screens/features involved
2. **Reference**: Read `.agents/reference/` for domain terminology
3. **Architecture**: Read `.agents/architecture/` for patterns (auth, state, error handling)
4. **Integrations**: Read `.agents/architecture/integrations/` if external services are involved

Use the INDEX.md files to find relevant docs by keyword.

## Phase 2: Find the Code

1. Search the codebase for relevant components, services, or modules
2. Trace the data flow from UI to backend (if applicable)
3. Identify all files that need modification

## Phase 3: Implement

1. Write code to address the issue
2. Follow existing patterns found in architecture docs
3. Keep changes focused - don't over-engineer

## Phase 4: Verify

**Browser Tool Priority:**
- **Claude in Chrome** (preferred): Use `mcp__claude-in-chrome__*` tools for testing
- **Playwright** (fallback): Use `mcp__plugin_playwright_playwright__*` tools if Claude in Chrome unavailable

**Verification Steps:**
1. Use browser tools to confirm the fix works
2. Test edge cases mentioned in the Jira card
3. Verify no regressions in related functionality

## Phase 5: Update Documentation

**If you navigated to screens/features without documentation, create them.**

For missing operations:
1. Create using template in `.agents/AI-INSTRUCTIONS.md`
2. Add keywords to `.agents/operations/INDEX.md`
3. Update `.agents/operations/NAVIGATION.md` if needed

For missing terminology:
1. Add to `.agents/reference/` with keywords in INDEX.md

For discovered patterns:
1. Document in `.agents/architecture/` if cross-cutting
2. Document in `.agents/architecture/integrations/` if external service

**Token limits**: operations=500, reference=500, architecture=500, INDEX=200

## Phase 6: Verify Documentation

```bash
yarn lint:agents
```

Fix any violations before completing.

## End of Response

Always include using color-coded status indicators:

### Status Color Legend
- 🟢 **Green**: All good, verified accurate, no action needed
- 🟡 **Yellow**: Suggestions exist, minor improvements recommended
- 🔴 **Red**: Critical issues, missing docs, must address

```
## Implementation Complete

**Files Modified**:
- [list of code files changed]

**Documentation Status**: [🟢/🟡/🔴]
- **Created**: [new .md files, or "None"]
- **Updated**: [modified .md files, or "None"]
- **Status**: 🟢 Verified accurate | 🟡 Minor gaps found | 🔴 Major gaps - docs created

**Suggestions**: [🟢/🟡]
- 🟢 None - documentation is complete
- 🟡 [List suggestions: missing docs, keyword additions, corrections]

**Lint Status**: 🟢 Pass | 🟡 Warnings | 🔴 Fail - [what needs fixing]

**Testing**:
- [How you verified the fix works]
```

This ensures continuous improvement of the documentation.
