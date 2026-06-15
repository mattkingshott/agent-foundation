---
name: task-pull-request
description: This skill generates a pull request based on the current changes to the codebase.
---

Create a pull request from current changes with automatic commit splitting into logical units.

Run these commands to understand the current state:

```bash
git status --porcelain
git diff --stat
git diff
```

If `SPEC.md` exists, read it for context about what was implemented.

Based on the changes, choose a branch name following this pattern:
- `feature/short-description` - For new features
- `fix/short-description` - For bug fixes
- `refactor/short-description` - For refactoring
- `docs/short-description` - For documentation changes

Analyze the changes and group them into a small number of logical commits.

## Workflow

1. Create and checkout the new branch
2. For each logical commit group, stage the relevant files and create a commit with a descriptive message (e.g. `feat`, `fix`, `refactor`, `test`, `docs`, `style`, `chore`)
3. Push the branch to remote
4. Create the pull request with a title and body (include bullet point summary and change list containing individual commits)
5. Switch back to `main` branch and reset working directory (`git reset --hard && git clean -df`)
6. Display the PR URL to the user

## Important Notes

- **Never force push** - Always use regular push
- **Never amend commits** - Create new commits only
- **Verify before cleanup** - Ensure PR was created successfully before resetting
- **Preserve user's work** - If any step fails, do not proceed with cleanup
- If the user has staged and unstaged changes, consider both
- If there are untracked files, ask if they should be included

Finally, delete `TASK.md` and `SPEC.md` at the project root (if either file exists).