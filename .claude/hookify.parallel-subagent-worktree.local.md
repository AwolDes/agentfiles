---
name: parallel-subagent-worktree
enabled: true
event: all
pattern: Task.*subagent|multiple.*agent|parallel.*agent|agent.*parallel
action: block
---

🛑 **Parallel Subagent Launch Detected**

Before launching multiple subagents in parallel, you must ask the user if they want to use **worktrees** for isolated work.

**Required action:**
Use `AskUserQuestionTool` to ask something like:
- "Would you like me to set up git worktrees for these parallel tasks?"
- "Should I use worktrees to isolate each agent's work?"

**Why worktrees help with parallel agents:**
- Each agent can work in an isolated directory
- Prevents file conflicts between concurrent edits
- Allows proper git operations without interference
- Cleaner separation of concerns

Only proceed with parallel subagents after confirming with the user about worktree usage.
