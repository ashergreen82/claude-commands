# Jira Investigation Protocol

You are investigating a Jira card. Your goal is to **understand the task thoroughly** and **update documentation** before writing any code.

## Rules

- **DO NOT write code** during this investigation
- **DO NOT suggest fixes** yet
- Focus entirely on understanding and documenting

## Phase 0: Verify Browser Tools (MANDATORY PREREQUISITE)

**⚠️ CRITICAL: This MUST be the first thing you do. Do not proceed to Phase 1 without completing this step.**

**Test browser tools immediately using this exact sequence:**

1. **Test Claude in Chrome first:**
   ```
   Call: mcp__claude-in-chrome__tabs_context_mcp with createIfEmpty=true
   ```
   - If successful → Browser tools are ready, proceed to Phase 1
   - If error → Continue to step 2

2. **Test Playwright as fallback:**
   ```
   Call: mcp__playwright__browser_snapshot
   ```
   - If successful → Browser tools are ready, proceed to Phase 1
   - If error → Continue to step 3

3. **Both tools failed - STOP IMMEDIATELY:**

   **Do NOT proceed. Output this exact message:**

   ```
   ❌ BROWSER TOOLS NOT AVAILABLE

   I cannot proceed with the Jira investigation because browser tools are required to observe and recreate the issue.

   Tested:
   - Claude in Chrome: [error message]
   - Playwright: [error message]

   To continue, please either:
   1. Fix the browser tool connection and tell me to retry
   2. Provide screenshots/recordings of the issue as an alternative
   3. Explicitly tell me to skip browser observation (not recommended)

   I will wait for your response before proceeding.
   ```

   **STOP HERE. Do not read documentation, do not parse the task, do not do anything else until the user responds.**

**Why this matters:**
- UI observation catches issues code analysis misses
- Recreating the bug ensures we're solving the right problem
- Screenshots document the actual vs expected behavior
- Skipping this step leads to incomplete understanding

## Phase 1: Parse the Task

1. Read the Jira card details the user provided (description, acceptance criteria, screenshots)
2. Identify the key screen/feature mentioned
3. Extract keywords for searching docs
4. Convert acceptance criteria to Gherkin format:
   ```gherkin
   Feature: [Feature name]

   Scenario: [Scenario description]
     Given [initial context]
     When [action taken]
     Then [expected outcome]
   ```

## Phase 2: Parallel Documentation & UI Exploration

**First: Get navigation context**
1. Quick read: `.agents/operations/NAVIGATION.md` to find the screen path

**Then: Spawn TWO subagents in parallel (use two Task tool calls in same message):**

### Subagent 1: Documentation Explorer
```
Task tool with subagent_type=Explore:
- Description: "Search documentation"
- Prompt: "Search .agents/ documentation for [feature/screen] related to [keywords from Phase 1].

  Check ALL of these:
  - AI-INSTRUCTIONS.md (doc structure)
  - product-overview.md (product context)
  - project-overview.md (repo role)
  - operations/INDEX.md (find relevant operations)
  - reference/INDEX.md (terminology)
  - architecture/INDEX.md (patterns like auth, state)
  - integrations/INDEX.md (external services)

  Return:
  - Relevant documentation files found
  - Existing vs missing operations
  - Relevant architecture patterns
  - Domain terminology needed"
- Thoroughness: "medium"
```

### Subagent 2: UI Explorer
```
Task tool with subagent_type=Explore:
- Description: "Explore UI and recreate issue"
- Prompt: "Use browser tools to explore the application and recreate the issue described in the Jira card.

  Navigation path (from NAVIGATION.md): [insert path here]

  Your tasks:
  1. Navigate to the target screen using the path above
  2. If you encounter screens you don't know how to navigate (like login), check `.agents/operations/[screen-name].md` for workflows
  3. Observe current behavior vs expected behavior
  4. Attempt to recreate the issue
  5. Take screenshots of key states
  6. Document what you observe

  SCOPE LIMIT: Only read .agents/operations/ folder for navigation help. Don't read architecture or reference docs.

  Return:
  - Whether you successfully recreated the issue
  - Screenshots and observations
  - Current vs expected behavior
  - Any additional findings"
- Thoroughness: "medium"
```

**Both subagents run simultaneously. When both return, synthesize their findings.**

## Phase 3: Review Subagent Findings

**Both subagents have returned their findings. Now synthesize:**

### From Documentation Subagent:
- Which operations exist vs missing?
- What architecture patterns are relevant?
- Any integration concerns?
- Domain terminology to understand?

### From UI Subagent:
- Was the issue successfully recreated? (🟢/🔴)
- What's the current vs expected behavior?
- Screenshots and observations
- Any additional findings?

### Your Job:
1. Identify gaps in understanding from both reports
2. Cross-reference: Do UI observations match what docs describe?
3. Note discrepancies between documented behavior and actual behavior
4. If UI subagent couldn't recreate the issue, investigate why (missing context? wrong path? issue resolved?)

**If the issue was NOT recreated successfully:**
- Review the UI subagent's findings
- You may need to manually explore using browser tools
- Or ask the user for clarification before proceeding

## Phase 4: Interview Developer

Use AskUserQuestion to fill gaps. Ask about:
- Unclear acceptance criteria
- Expected vs actual behavior
- Edge cases or related features
- Any context not in the Jira card

Keep questions focused and batched (2-3 at a time max).

## Phase 5: Create Investigation Summary

**Create `.agents/investigations/[TICKET-ID].md` with all findings:**

```markdown
# Investigation: [TICKET-ID] - [Brief Title]

**Date**: [YYYY-MM-DD]
**Investigator**: Claude
**Status**: Ready for Implementation

## Acceptance Criteria (Gherkin)

Feature: [Feature name]

Scenario: [Primary scenario]
  Given [initial context]
  When [action taken]
  Then [expected outcome]

[Additional scenarios as needed]

## Browser Investigation

**Tools Used**: [Claude in Chrome | Playwright | Skipped with permission]

**Bug Recreation**: [🟢 Success | 🔴 Failed]
- [Description of what you observed]
- [Screenshots references if applicable]

**Observed Behavior**:
- Current: [what actually happens]
- Expected: [what should happen]

## Navigation Path

Login → [operation chain] → Target Screen

**Operations Consulted**:
- `.agents/operations/[file].md` - [brief description]
- [Additional files...]

## Root Cause Analysis

[Technical explanation of what's broken and why, including:]
- Component/service affected
- Data flow issue
- State management problem
- API/integration issue
- etc.

## Implementation Plan

**Files to Modify**:
- `path/to/file1.ts` - [what needs to change]
- `path/to/file2.tsx` - [what needs to change]

**Approach**:
1. [Step 1]
2. [Step 2]
3. [Step 3]

**Edge Cases to Consider**:
- [Edge case 1]
- [Edge case 2]

## Interview Notes

**Q**: [Question asked]
**A**: [User response]

[Additional Q&A as needed]

## Testing Strategy

- [ ] Verify fix resolves the reported issue
- [ ] Test edge cases listed above
- [ ] Check for regressions in related features
- [ ] [Additional test scenarios]
```

## Phase 6: Checkpoint

Report to user - keep it terse and scannable:

```
## Investigation Complete ✅

**File**: `.agents/investigations/[TICKET-ID].md`

**Bug Recreation**: [🟢 Success | 🔴 Failed] - [one sentence]

**Root Cause**: [one sentence technical explanation]

---

**Next Steps**:
• Reply "proceed" to implement now
• Reply "/clear" then paste below for fresh context

**Copy for fresh context**:
```
/implement

Investigation: .agents/investigations/[TICKET-ID].md
```
```

Do not proceed to code until the developer confirms.
