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

## Phase 2: Search Documentation

1. Read `.agents/AI-INSTRUCTIONS.md` - understand the documentation structure
2. Read `.agents/product-overview.md` - understand the product and terminology
3. Read `.agents/project-overview.md` - understand this repo's role in the system
4. Read `.agents/operations/INDEX.md` - search for relevant keywords
5. Read `.agents/reference/INDEX.md` - search for relevant terminology
6. Read `.agents/operations/NAVIGATION.md` - find the path to the screen
7. Read `.agents/architecture/INDEX.md` - check for relevant patterns (auth, state, etc.)
8. Read `.agents/architecture/integrations/INDEX.md` - check if external services are involved
9. Load each operation in the navigation path (login → patient query → target screen)
10. Note which operations exist vs are missing

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

## Phase 5: Update Documentation

**Token limits**: operations=500, reference=500, architecture=500, INDEX=200. Keep docs concise.

For any screen you explored that lacks documentation:
1. Create a new operation file using the template in `.agents/AI-INSTRUCTIONS.md`
2. Add keywords to `.agents/operations/INDEX.md`
3. Update `.agents/operations/NAVIGATION.md` if hierarchy changed

For existing operations with gaps:
1. Add missing steps or gotchas
2. Correct any inaccuracies you observed

For terminology discovered:
1. Add to `.agents/reference/` if it's reusable domain knowledge
2. Update `.agents/reference/INDEX.md` with keywords

For patterns or integrations discovered:
1. Document in `.agents/architecture/` if it's a cross-cutting concern
2. Document in `.agents/architecture/integrations/` if it involves external services

## Phase 6: Verify Documentation

Run linters to ensure compliance:
```bash
yarn lint:agents
```

Fix any structure or token limit violations before proceeding.

## Phase 7: Checkpoint

When investigation is complete, report using color-coded status indicators:

### Status Color Legend
- 🟢 **Green**: All good, verified accurate, no action needed
- 🟡 **Yellow**: Suggestions exist, minor improvements recommended
- 🔴 **Red**: Critical issues, missing docs, must address before proceeding

```
## Investigation Complete

**Jira Summary**: [One sentence summary of the task]

**Browser Tools Used**: [Claude in Chrome | Playwright | Skipped with permission]

**Bug Recreation**: [🟢/🔴]
- 🟢 Successfully recreated the issue - [brief description of what you observed]
- 🔴 Could not recreate - [explain why and what you found instead]

**Screen Path**: Login → [operation chain] → Target Screen

**Operations Consulted**:
- [list of .md files you read]

**Documentation Status**: [🟢/🟡/🔴]
- **Created**: [new .md files, or "None"]
- **Updated**: [modified .md files, or "None"]
- **Status**: 🟢 Verified accurate | 🟡 Minor gaps found | 🔴 Major gaps - docs created

**Suggestions**: [🟢/🟡]
- 🟢 None - documentation is complete
- 🟡 [List suggestions: missing docs, keyword additions, corrections]

**Lint Status**: 🟢 Pass | 🟡 Warnings | 🔴 Fail - [what needs fixing]

**Understanding**:
[2-3 sentences explaining the issue and what needs to happen]

**Ready to implement?**
- Reply "proceed" to start implementation
- Reply "refine" to continue investigating
- Reply "/clear" then "/implement" for fresh context implementation
```

Do not proceed to code until the developer confirms.
