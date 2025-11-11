---
description: Create a pull request with conventional commit format
model: haiku
---

Create a pull request for the current branch. Accepts optional target branch as parameter (defaults to main).

Usage:
- `/pr` - creates PR to main
- `/pr develop` - creates PR to develop

General Rules:
- Branch names always start with Jira ticket (e.g., `OR-1343-feature-description`)
- Extract Jira ticket from branch name for PR description
- PR title should use conventional commit format
- Include only context explanation in PR description (no Summary section)
- End PR description with `Jira: [TICKET-ID]`
- Never include co-authored lines or Claude attribution
- Never merge a PR unless it is asked
- **NEVER create, update, or push to PRs unless explicitly requested by the user**

Steps:
1. Run `git branch --show-current` to get the current branch name
2. Extract Jira ticket from branch name if present (format: TICKET-123/description)
3. Run `git log origin/main..HEAD --oneline` (or use provided target branch) to see commits
4. Run `git log origin/main..HEAD --format="%s%n%b"` to get full commit messages
5. Analyze commits:
   - If only ONE commit: use its title and body as-is for PR
   - If MULTIPLE commits:
     - Use first commit title as base
     - Enhance description by combining interesting details from all commits
     - Keep conventional commit format
6. Create PR title following conventional commit format:
   - `type(scope): description`
   - Types: feat, fix, refactor, test, docs, chore, style, perf, ci, build
7. Create PR body:
   - Start with detailed description
   - If Jira ticket found, add footer: `Jira: TICKET-123`
   - Do NOT add co-authored lines or Claude attribution
8. Use `gh pr create` to create the pull request

PR Format:
```
Title: type(scope): description

Body:
Detailed explanation of changes and context.

If multiple commits, include relevant details from all of them.

Jira: TICKET-123
```

Important:
- Target branch defaults to `main` if not specified
- Follow conventional commit format strictly
- Extract ticket from branch name pattern: `[A-Z]+-[0-9]+` before first `/`
- Use `gh pr create --title "..." --body "..." --base <target-branch>`
- If only one commit, preserve its exact title and description
- If multiple commits, intelligently combine them
