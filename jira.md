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

## Phase 2: Search Documentation

**Use Task tool with Explore agent for efficient context management:**

```
Task tool with subagent_type=Explore:
- Prompt: "Search documentation for [feature/screen] related to [keywords from Phase 1].
  Check: AI-INSTRUCTIONS.md, product-overview.md, project-overview.md, operations/INDEX.md,
  reference/INDEX.md, operations/NAVIGATION.md, architecture/INDEX.md, integrations/INDEX.md.
  Find navigation path, existing operations, and relevant patterns."
- Thoroughness: "medium"
```

**The Explore agent will compress findings and return:**
- Relevant documentation files
- Navigation path to target screen
- Existing vs missing operations
- Relevant architecture patterns

**Synthesize the results into your understanding.**

## Phase 3: Explore the UI (REQUIRED)

**This phase is vital - you must observe and recreate the issue using browser tools.**

**You verified browser tools in Phase 0. Now use them to explore:**

**Browser Tool Priority:**
- **Claude in Chrome** (if available): Use `mcp__claude-in-chrome__screenshot`, `mcp__claude-in-chrome__read_page`, etc.
- **Playwright** (fallback): Use `mcp__playwright__browser_snapshot`, `mcp__playwright__browser_navigate`, etc.

**Steps:**
1. Start the application (navigate to the local dev URL or production URL)
2. See the current page state using browser tools
3. Navigate to the relevant screen following the navigation path from Phase 2
4. Follow the operation steps to recreate the user's exact path
5. Observe the current behavior - what does the UI actually show?
6. **Attempt to recreate the issue** described in the Jira card
7. Document what you observe vs what the Jira card describes
8. Take screenshots of key states (before, during, after)

**Required Output:**
- Confirm you successfully recreated the bug (or explain why you couldn't)
- Screenshots showing the actual vs expected behavior
- Any additional observations not mentioned in the Jira card

If you cannot recreate the issue, ask the user for clarification before proceeding.

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

Report to user using color-coded status indicators:

### Status Color Legend
- 🟢 **Green**: All good, verified accurate, no action needed
- 🟡 **Yellow**: Suggestions exist, minor improvements recommended
- 🔴 **Red**: Critical issues, missing docs, must address before proceeding

```
## Investigation Complete

**Investigation File**: `.agents/investigations/[TICKET-ID].md` ✅ Created

**Jira Summary**: [One sentence summary of the task]

**Bug Recreation**: [🟢/🔴]
- 🟢 Successfully recreated - [brief description]
- 🔴 Could not recreate - [explanation]

**Root Cause**: [One sentence technical explanation]

**Files to Modify**: [count] files identified

**Ready to implement?**
- Reply "proceed" or "/implement" to start implementation
- Reply "refine" to continue investigating
```

Do not proceed to code until the developer confirms.
