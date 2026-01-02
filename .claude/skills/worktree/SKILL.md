---
name: worktree
description: Create and manage git worktrees for parallel development. Use when the user wants to create a worktree, work on a feature in isolation, set up a new branch with its own directory, or manage existing worktrees.
allowed-tools: Bash(bin/worktree:*), Bash(git worktree:*)
---

# Git Worktree Management

Manage git worktrees using the `bin/worktree` script for parallel development workflows.

## Available Commands

```bash
bin/worktree list              # Show all worktrees
bin/worktree create <branch>   # Create worktree for branch
bin/worktree delete <branch>   # Remove worktree
bin/worktree go <branch>       # Print path (for cd)
```

## Naming Convention

Worktrees are created as sibling directories to the main repo. Slashes become dashes:

| Branch | Directory |
|--------|-----------|
| `feature/auth` | `../jolly-blitzen-feature-auth` |
| `bugfix/login` | `../jolly-blitzen-bugfix-login` |
| `experiment/new-ui` | `../jolly-blitzen-experiment-new-ui` |

## Instructions

### When creating a worktree:

1. If user provides a branch name (contains `/`), use it directly
2. If user provides a task description, convert to `feature/<kebab-case-description>`
3. Run `bin/worktree create <branch>`
4. Tell the user the path and how to switch: `cd <path>`

### When listing worktrees:

Run `bin/worktree list` and present the results.

### When deleting a worktree:

1. Confirm the branch name with the user if ambiguous
2. Run `bin/worktree delete <branch>`

## Examples

**User**: "Create a worktree for adding user notifications"
```bash
bin/worktree create feature/user-notifications
```

**User**: "Set up a worktree for branch bugfix/login-issue"
```bash
bin/worktree create bugfix/login-issue
```

**User**: "Show me my worktrees"
```bash
bin/worktree list
```

**User**: "Remove the notifications worktree"
```bash
bin/worktree delete feature/user-notifications
```
