---
description: Create conventional commit with optional push
model: haiku
---

Create a conventional commit and optionally push the changes to the remote repository.

Steps:
1. Run `git status` to see current changes
2. Run `git diff` to see the actual changes
3. Run `git branch --show-current` to get the current branch name
4. Extract Jira ticket from branch name if present (format: TICKET-123/description)
5. Analyze the changes and create a conventional commit message following the format:
   - `type(scope): description`
   - Types: feat, fix, refactor, test, docs, chore, style, perf, ci, build
   - Keep description concise and in imperative mood
   - Do NOT add co-authored lines or Claude attribution
   - If Jira ticket found in branch name, add footer: `Ticket: TICKET-123`
6. Stage all changes with `git add .`
7. Create commit using the format from CLAUDE.md git commit rules
8. Parse flags from the command arguments:
   - Check for push flags: `--push` or `-p` or combined in flags like `-pf`
   - Check for force flags: `--force` or `-f` or combined in flags like `-pf`
   - If both push and force flags are present, push with `git push --force-with-lease`
   - If only push flag is present, push normally with `git push`

Commit message format:
```
type(scope): description

Ticket: TICKET-123
```

Important:
- Use ONLY the `-m` flag for commit messages (no additional descriptions)
- NO --no-verify or other flags unless explicitly requested
- Follow conventional commit format strictly
- Keep message concise and clear
- Extract ticket from branch name pattern: `[A-Z]+-[0-9]+` before first `/`
- Never include co-authored lines or Claude attribution
- By default only commit, use `--push` or `-p` flag to also push to remote
- Add `--force` or `-f` flag along with push flags to force push with lease
- Flags can be combined: `-pf` combines push and force flags
- Usage:
  - `/commit` (commit only)
  - `/commit --push` or `/commit -p` (commit and push)
  - `/commit -p -f` or `/commit --push --force` (commit and force push with lease)
  - `/commit -pf` (commit and force push with lease using combined flags)