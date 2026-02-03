# Jira Investigation Protocol

You are investigating a Jira card. Your goal is to **understand the task thoroughly** and **update documentation** before writing any code.

## Rules

- **DO NOT write code** during this investigation
- **DO NOT suggest fixes** yet
- Focus entirely on understanding and documenting

## CRITICAL: Browser Tools Required

**Before starting, verify browser tools are available using this priority order:**

1. **Prefer Claude in Chrome**: Try `mcp__claude-in-chrome__tabs_context_mcp` first
2. **Fallback to Playwright**: If Claude in Chrome is unavailable, use `mcp__plugin_playwright_playwright__browser_snapshot`

If both browser tools are unavailable or return an error:
1. **STOP immediately**
2. Inform the user: "Browser tools are not connected. I need browser access to observe and recreate the issue."
3. Wait for the user to either:
   - Fix the connection and tell you to continue
   - Provide an alternative (screenshots, recordings, etc.)
   - Tell you to skip browser observation

**Do not proceed past Phase 2 without browser access or explicit user permission to skip.**

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

**Browser Tool Priority:**
- **Claude in Chrome** (preferred): Use `mcp__claude-in-chrome__screenshot`, `mcp__claude-in-chrome__read_page`, etc.
- **Playwright** (fallback): Use `mcp__plugin_playwright_playwright__browser_snapshot`, etc.

**Steps:**
1. See the current page state using browser tools
2. Navigate to the relevant screen
3. Follow the operation steps to recreate the user's exact path
4. Observe the current behavior - what does the UI actually show?
5. **Attempt to recreate the issue** described in the Jira card
6. Document what you observe vs what the Jira card describes
7. Take screenshots if helpful

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
