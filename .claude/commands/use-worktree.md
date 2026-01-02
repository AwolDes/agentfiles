# /use-worktree - Git Worktree Command

Create or manage git worktrees via `bin/worktree`.

## Arguments: $ARGUMENTS

## Action

Parse `$ARGUMENTS` and execute:

| Input | Action |
|-------|--------|
| Empty | Run `bin/worktree list`, ask what to do next |
| `list` | Run `bin/worktree list` |
| `delete <branch>` | Run `bin/worktree delete <branch>` |
| Branch name (has `/`) | Run `bin/worktree create <branch>` |
| Task description | Run `bin/worktree create feature/<kebab-case-description>` |

## Examples

- `/worktree` → lists worktrees
- `/worktree feature/auth` → creates worktree for `feature/auth`
- `/worktree user notifications` → creates worktree for `feature/user-notifications`
- `/worktree delete feature/old` → deletes the worktree

After creating, tell the user how to switch: `cd <path>`
