# Audit and Migrate Agent Documentation

You are auditing the existing `.agents/` documentation structure and migrating it to the standard format.

**⚠️ IMPORTANT**: This skill is synchronized with `init-docs.md`. Any changes to structure, templates, or standards here MUST be reflected in `init-docs.md`. Review both files when making updates.

## Phase 1: Scan Current Structure

Analyze the `.agents/` directory and identify:

1. **Non-standard files at root** - Files other than:
   - AI-INSTRUCTIONS.md
   - product-overview.md
   - project-overview.md

2. **Non-standard folders at root** - Folders other than:
   - operations/
   - reference/
   - architecture/
   - scripts/
   - .scratch/

3. **Nested documentation structures** - Look for:
   - `docs/` subdirectory with nested folders
   - `technical/`, `product/`, `infrastructure/`, `quickstart/`, etc.
   - Any multi-level folder hierarchies

4. **Missing standard structure** - Check if standard folders/files exist

Create a summary of findings with:
- What exists currently
- What doesn't match the standard
- Estimated content to migrate (number of .md files)

## Phase 2: Interview User About Migrations

For each non-standard element found, use `AskUserQuestion` to ask:

### For infrastructure/devops/deployment docs:
"Found infrastructure documentation (e.g., NGINX.md, PM2.md, SSL.md). Where should these go?"
- Options:
  - architecture/devops/ (Create devops subdirectory under architecture)
  - architecture/integrations/ (Treat as external service integrations)
  - architecture/ root (Create individual architecture docs)
  - Delete (Content is redundant)

### For quickstart/setup/getting-started docs:
"Found quickstart/setup documentation. Where should these go?"
- Options:
  - architecture/deployment.md and architecture/setup.md (Separate files)
  - reference/setup.md (Treat as reference material)
  - Delete (Redundant with root README.md or DEPLOYMENT.md)

### For product/domain/business docs:
"Found product/domain documentation. Where should these go?"
- Options:
  - Merge into product-overview.md (Consolidate product context)
  - reference/ (Create reference docs for domain concepts)
  - Delete (Content is redundant)

### For technical/api/database docs:
"Found technical documentation (API, database, etc.). Where should these go?"
- Options:
  - architecture/ (Create architecture docs)
  - reference/ (Create reference docs for schemas, APIs)
  - operations/ (If they describe operational procedures)
  - Delete (Redundant)

### For decision/ADR docs:
"Found decision or ADR documentation. Where should these go?"
- Options:
  - architecture/decisions/ (Keep decisions under architecture)
  - Delete (Decisions are outdated or captured elsewhere)

### For the AI-INSTRUCTIONS.md file:
"Found existing AI-INSTRUCTIONS.md. What should we do?"
- Options:
  - Keep existing (Preserve current content)
  - Replace with standard template (Use init-docs template)
  - Merge (Combine existing content with standard template)
  - Enhance with lazy loading policy (Add critical documentation loading rules)

If user chooses "Enhance with lazy loading policy", ensure the AI-INSTRUCTIONS.md has this at the top:

```markdown
## ⚡ CRITICAL: Documentation Loading Policy

**MINIMIZE CONTEXT USAGE. MAXIMIZE REASONING SHARPNESS.**

### Core Principle: Lazy Loading Only

**DO NOT pre-load documentation files.** Only read specific docs when the task explicitly requires domain/product/architecture knowledge.

### Default Mode: Work Directly with Code

For **90% of tasks** (bug fixes, features, styling, refactoring):
1. ✅ Use search/grep to find relevant code
2. ✅ Read only specific files you need
3. ❌ **DO NOT read any documentation files**

### When to Load Documentation (Rare)

**Read docs ONLY for these scenarios:**

- **User questions** → `product-overview.md` or `reference/`
- **Architecture decisions** → `architecture/`
- **Deployment issues** → `architecture/devops/TROUBLESHOOT.md`
- **Setup questions** → `architecture/setup.md`

### How to Decide
Ask: **"Can I complete this task by reading only code files?"**
- ✅ YES → Skip all documentation
- ❌ NO → Read ONE specific doc file

### Examples
- "Fix button styling" → Read code only
- "Add delete feature" → Read code only
- "Should we add X feature?" → Read `product-overview.md`
- "Deployment failed" → Read `architecture/devops/TROUBLESHOOT.md`
```

## Phase 3: Check for Content Redundancy

Before migrating, check if content is redundant with root-level files:

1. **Compare `.agents/docs/quickstart/DEPLOYMENT.md` with root `DEPLOYMENT.md`**
   - Root = Initial setup guide → Keep
   - .agents = Ongoing workflow → Migrate to architecture/deployment.md

2. **Compare `.agents/docs/technical/DATABASE.md` with root `server/src/models/database.sql`**
   - If .agents doc just describes what's in the .sql file → Can delete
   - If it has additional context → Migrate to reference/database-schema.md

3. **Compare `.agents/docs/infrastructure/` with root `INFRASTRUCTURE.md`**
   - Root = High-level overview → Keep
   - .agents = Detailed configs → Migrate to architecture/devops/

Report findings to user and ask for confirmation before deleting any files.

## Phase 4: Execute Migrations

Based on user answers, perform the migrations:

1. **Create new directories if needed** (e.g., `architecture/devops/`)

2. **Copy/move files** to their new locations:
   ```bash
   cp old/path/FILE.md new/path/file.md
   ```

3. **Update INDEX.md files** in operations/, reference/, and architecture/ with keywords for new docs

4. **Update AI-INSTRUCTIONS.md** with lazy loading policy if user requested

5. **Remove old structure** after confirming migration success:
   ```bash
   rm -rf .agents/docs/
   ```

## Phase 5: Update INDEX Files

For each migrated file, add an entry to the appropriate INDEX.md:

**Example for architecture/INDEX.md:**
```markdown
**Keywords**: deployment, ci/cd, github actions, production
- [Deployment Guide](./deployment.md)

**Keywords**: nginx, pm2, ssl, troubleshooting
- [DevOps Documentation](./devops/INDEX.md)
```

**Example for architecture/devops/INDEX.md:**
```markdown
**Keywords**: ci/cd, github actions, deploy, pipeline
- [CI/CD Guide](./CICD.md)

**Keywords**: nginx, reverse proxy, ssl
- [NGINX Configuration](./NGINX.md)

**Keywords**: pm2, process manager, restart
- [PM2 Guide](./PM2.md)
```

## Phase 6: Validate

Run validation checks:

```bash
npm run lint:structure  # Check folder structure
npm run lint:docs       # Check token limits
```

If any files exceed token limits, inform the user and suggest:
- Split large files into multiple focused docs
- Move detailed content to root-level docs
- Remove redundant content

## Phase 7: Report

Provide a migration summary:

```markdown
## Documentation Audit Complete

### Migrated Content
- X files moved from docs/infrastructure/ → architecture/devops/
- X files moved from docs/quickstart/ → architecture/
- X files consolidated into product-overview.md

### Structure Changes
- Created architecture/devops/
- Updated 3 INDEX.md files with new keywords
- Enhanced AI-INSTRUCTIONS.md with lazy loading policy

### Validation
- ✅ Structure: All checks passed
- ✅ Token limits: X files, all within limits
- ✅ Total: XXXX tokens

### Removed
- docs/ (old nested structure)
- X redundant files

### Next Steps
1. Review migrated content for accuracy
2. Commit changes: git add .agents/ && git commit -m "Migrate to standard .agents/ structure"
3. Update any external links to old documentation paths
```

## Important Notes

- **Never delete files without user confirmation**
- **Always preserve original content** - migration should not lose information
- **Check redundancy** - avoid duplicating content that exists elsewhere
- **Respect token limits** - if a file exceeds 500 tokens, suggest splitting
- **Update references** - check if other files link to old paths and update them
- **Preserve git history** - use `git mv` if files are already tracked

## Error Handling

If issues occur during migration:
1. Stop the migration
2. Report what was completed
3. Explain what failed and why
4. Ask user how to proceed
5. Do not leave the structure in a half-migrated state
