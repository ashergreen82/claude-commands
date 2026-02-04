# Implementation Mode

Documentation has been reviewed. Now implement the fix.

## Phase 1: Load Investigation Summary

**CRITICAL: Start by reading the investigation file created by `/jira` command.**

1. **Read `.agents/investigations/[TICKET-ID].md`** - This contains:
   - Acceptance criteria (Gherkin format)
   - Bug recreation details
   - Root cause analysis
   - Files to modify
   - Implementation plan
   - Edge cases and testing strategy

2. **If investigation file is missing:**
   - Ask user for the ticket ID
   - Or request they run `/jira` first to investigate
   - Do NOT proceed without understanding the context

3. **Supplement with architecture docs only if needed:**
   - **Operations**: `.agents/operations/` for screen-specific steps
   - **Reference**: `.agents/reference/` for domain terminology
   - **Architecture**: `.agents/architecture/` for patterns (auth, state, error handling)
   - **Integrations**: `.agents/architecture/integrations/` for external services

## Phase 2: Find the Code

1. Search the codebase for relevant components, services, or modules
2. Trace the data flow from UI to backend (if applicable)
3. Identify all files that need modification

## Phase 3: Implement

1. Write code to address the issue
2. Follow existing patterns found in architecture docs
3. Keep changes focused - don't over-engineer

## Phase 4: Verify

**Use browser tools to test the implementation:**

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

## Phase 5: Update Investigation File

**Add implementation results to `.agents/investigations/[TICKET-ID].md`:**

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

### Testing Results

**Gherkin Scenarios**: [🟢 All Pass | 🟡 Partial | 🔴 Failed]

Scenario: [Scenario name]
  Given [context]
  When [action]
  Then [outcome]
  **Result**: [🟢 Pass | 🔴 Fail] - [explanation]

**Edge Cases**: [🟢 All Pass | 🟡 Partial | 🔴 Failed]
- [Edge case 1]: [🟢 Pass | 🔴 Fail] - [details]
- [Edge case 2]: [🟢 Pass | 🔴 Fail] - [details]

**Regressions**: [🟢 None | 🔴 Found]
- [Description of any regressions found]

### Screenshots

[References to any before/after screenshots taken during verification]

### Notes

[Any additional observations, gotchas, or follow-up items]
```

**This creates a complete audit trail from investigation → implementation.**

## Phase 6: Final Report

Always include using color-coded status indicators:

### Status Color Legend
- 🟢 **Green**: All good, verified accurate, no action needed
- 🟡 **Yellow**: Minor issues, improvements recommended
- 🔴 **Red**: Critical issues, must address

```
## Implementation Complete ✅

**Investigation File**: `.agents/investigations/[TICKET-ID].md` - Updated with results

**Files Modified**:
- [list of code files changed with brief description]

**Testing Status**: [🟢/🟡/🔴]
- **Gherkin Scenarios**: [🟢 All Pass | 🟡 Partial | 🔴 Failed]
- **Edge Cases**: [🟢 All Pass | 🟡 Partial | 🔴 Failed]
- **Regressions**: [🟢 None | 🔴 Found]

**Overall Status**: [🟢/🟡/🔴]
- 🟢 Ready to commit - all tests pass
- 🟡 Ready with notes - minor issues documented
- 🔴 Needs attention - [what needs fixing]

**Next Steps**:
- Review changes and commit if satisfied
- Address any 🟡 or 🔴 items if needed
- Close the Jira ticket
```
