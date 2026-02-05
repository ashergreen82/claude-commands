# Audit and Migrate Agent Documentation

This command has been split into focused subcommands for better context management.

## Available Commands

| Command | Purpose |
|---------|---------|
| `/audit-scan` | Scan `.agents/` structure and report findings |
| `/audit-migrate` | Interview user, plan migrations, execute changes |
| `/audit-validate` | Run lint scripts and verify structure |

## Recommended Workflow

```
1. /audit-scan      → See what needs migration
2. /audit-migrate   → Plan and execute changes
3. /audit-validate  → Verify everything is correct
```

## Quick Audit (All-in-One)

If you want to run the full audit flow in one go, I'll orchestrate the subcommands:

1. First, run `/audit-scan` to analyze the current structure
2. Review the findings
3. If migrations needed, run `/audit-migrate`
4. Finally, run `/audit-validate` to confirm

Would you like me to start with `/audit-scan`?
