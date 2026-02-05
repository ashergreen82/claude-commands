# Implementation Mode

Documentation has been reviewed. Now implement the fix.

## Phase 1: Load Investigation Summary

**CRITICAL: Read the investigation file created by `/jira` command.**

**If user provides investigation file path:**
- Read that file immediately: `.agents/investigations/[TICKET-ID].md`

**If no path provided:**
- Ask user: "Which investigation file should I read? (e.g., `.agents/investigations/TICKET-123.md`)"
- Or ask for ticket ID to construct the path

**Investigation file contains everything you need:**
- Acceptance criteria (Gherkin format)
- Bug recreation details
- Root cause analysis
- Files to modify
- Implementation plan
- Edge cases and testing strategy

**Only supplement with other docs if needed:**
- `.agents/operations/` - screen-specific workflows (for UI-related processes), endpoints (for API's), etc.
- `.agents/architecture/` - patterns (auth, state, error handling)
- `.agents/reference/` - domain terminology

## Phase 2: Find the Code

1. Search the codebase for relevant components, services, or modules
2. Trace the data flow from UI to backend (if applicable)
3. Identify all files that need modification

## Phase 3: Write Failing Tests

**Write automation tests FIRST before any implementation code.**

**Test Priority (agent decides based on change scope):**
1. **E2E/Acceptance tests first** - Test complete user workflows
2. **Integration tests second** - Test component interactions
3. **Unit tests last** - Test isolated functions/methods

**Requirements:**
- Tests should fail initially (proving they test the actual bug/feature)
- Cover acceptance criteria from investigation file (Gherkin scenarios)
- Include edge cases identified in investigation file
- Follow existing test patterns in the codebase

## Phase 4: Implement & Verify Via Tests

**Iterative TDD cycle - repeat until tests pass (max 3 iterations):**

**Each Iteration:**
1. Write/modify implementation code to make tests pass
2. Follow existing patterns from architecture docs
3. Keep changes focused - don't over-engineer
4. Run tests and check results

**Iteration Tracking:**
- **Iteration 1-3**: If tests fail, analyze failures and repeat Phase 4
- **After 3 failed iterations**: Stop and ask user for guidance - something fundamental may be wrong with the approach

**Success Criteria:**
- All tests written in Phase 3 pass ✅
- No new test failures introduced
- Code follows existing patterns

## Phase 5: Verify Via Browser (UI Projects Only)

**IMPORTANT: Only applicable for projects with a user interface.**

**Trigger Check:**
- If modified files include frontend extensions (`.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, etc.)
- Use AskUserQuestion to confirm: "Should I verify this implementation in the browser?"
- If user says no or this is a backend-only change, skip to Phase 6

**Browser Tool Priority:**
- **Claude in Chrome** (preferred): Use `mcp__claude-in-chrome__*` tools for testing
- **Playwright** (fallback): Use `mcp__playwright__*` tools if Claude in Chrome unavailable

**Verification Steps:**
1. Navigate to the relevant screen and reproduce the original issue path
2. Verify the fix resolves the reported issue (check Gherkin scenarios from investigation file)
3. Test edge cases listed in the investigation file
4. Verify no regressions in related functionality
5. Take screenshots of before/after if helpful

**Document test results for investigation file update.**

## Phase 6: Update Investigation File

**Add implementation results to the investigation file (same file you read in Phase 1):**

Append to the end of the file:

```markdown
---

## Implementation Results

**Date**: [YYYY-MM-DD]
**Implementer**: Claude

### Files Modified

- `path/to/file1.ts` - [what was changed]
- `path/to/file2.tsx` - [what was changed]

### Changes Made

1. [Change 1 description]
2. [Change 2 description]
3. [Change 3 description]

### Automation Test Results

**Tests Written**:
- [Number] E2E/Acceptance tests
- [Number] Integration tests
- [Number] Unit tests

**Test Status**: [🟢 All Pass | 🔴 Failed]
- **Iterations Required**: [1-3]
- **Test Files**: `path/to/test1.spec.ts`, `path/to/test2.test.ts`

### Browser Verification Results (if applicable)

**Gherkin Scenarios**: [🟢 All Pass | 🟡 Partial | 🔴 Failed | N/A]

Scenario: [Scenario name]
  Given [context]
  When [action]
  Then [outcome]
  **Result**: [🟢 Pass | 🔴 Fail] - [explanation]

**Edge Cases**: [🟢 All Pass | 🟡 Partial | 🔴 Failed | N/A]
- [Edge case 1]: [🟢 Pass | 🔴 Fail] - [details]
- [Edge case 2]: [🟢 Pass | 🔴 Fail] - [details]

**Regressions**: [🟢 None | 🔴 Found | N/A]
- [Description of any regressions found]

### Screenshots

[References to any before/after screenshots taken during verification]

### Notes

[Any additional observations, gotchas, or follow-up items]
```

**This creates a complete audit trail from investigation → implementation.**

## Phase 7: Documentation Review

**Run `/doc-update` to check if `.agents/` documentation needs updates based on this implementation.**

If suggestions are returned, ask the user which (if any) to implement before finalizing. If no suggestions, proceed to Final Report.

## Phase 8: Final Report

Always include using color-coded status indicators:

### Status Color Legend
- 🟢 **Green**: All good, verified accurate, no action needed
- 🟡 **Yellow**: Minor issues, improvements recommended
- 🔴 **Red**: Critical issues, must address

```
## Implementation Complete ✅

**Investigation File**: [filename] - Updated with results

**Files Modified**:
- [list of code files changed with brief description]

**Automation Test Results**: [🟢/🟡/🔴]
- **E2E/Acceptance Tests**: [🟢 All Pass | 🔴 Failed]
- **Integration Tests**: [🟢 All Pass | 🔴 Failed]
- **Unit Tests**: [🟢 All Pass | 🔴 Failed]
- **Iterations Required**: [1-3]

**Browser Verification** (if applicable): [🟢/🟡/🔴/N/A]
- **Gherkin Scenarios**: [🟢 All Pass | 🟡 Partial | 🔴 Failed | N/A]
- **Edge Cases**: [🟢 All Pass | 🟡 Partial | 🔴 Failed | N/A]
- **Regressions**: [🟢 None | 🔴 Found | N/A]

**Overall Status**: [🟢/🟡/🔴]
- 🟢 Ready to commit - all tests pass
- 🟡 Ready with notes - minor issues documented
- 🔴 Needs attention - [what needs fixing]

**Next Steps**:
- Review changes and commit if satisfied
- Address any 🟡 or 🔴 items if needed
- Close the Jira ticket
```