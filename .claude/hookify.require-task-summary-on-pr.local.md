---
name: require-task-summary-on-pr
enabled: true
event: bash
pattern: gh\s+pr\s+create(?!.*--draft)
action: warn
---

📋 **Task Summary Required Before PR**

Before creating this pull request for review, you must:

1. Run `/task-summary` to summarize the work done
2. Commit the task summary to the branch

This ensures all PRs ready for review have proper documentation of the changes made.

**Next steps:**
1. Cancel this PR creation
2. Run `/task-summary` command
3. Commit the generated summary
4. Then create the PR

Note: Draft PRs (`gh pr create --draft`) do not require a task summary.
