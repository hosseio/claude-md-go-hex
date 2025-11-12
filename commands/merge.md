---
description: Merge or rebase another branch into the current branch with intelligent conflict analysis and resolution
tags: [project, gitignored]
---

You are tasked with merging or rebasing another branch into the current branch with intelligent conflict analysis and interactive resolution.

## Usage:
- `/merge main` - merge main into current branch (auto-pulls incoming branch first)
- `/merge main -r` - rebase current branch onto main (auto-pulls incoming branch first)
- `/merge -r main` - rebase current branch onto main (auto-pulls incoming branch first)
- `/merge main --rebase` - rebase current branch onto main (auto-pulls incoming branch first)
- `/merge main --no-pull` - skip pulling incoming branch before merge
- `/merge main --no-commit` - resolve conflicts but don't finalize merge (for manual review)
- `/merge main -r --no-commit` - rebase and stop before continuing (for manual review)

## Steps to follow:

### 1. Validate input:
   - Parse arguments to detect branch name and flags
   - Check for `-r` or `--rebase` flag in arguments
   - Check for `--no-pull` flag (default: false, will pull)
   - Check for `--no-commit` flag (default: false, will finalize)
   - Ensure a branch name was provided
   - Verify the branch exists locally or remotely
   - Set operation mode: MERGE or REBASE
   - Set pull mode: PULL (default) or NO_PULL
   - Set commit mode: AUTO_COMMIT (default) or NO_COMMIT

### 2. Pull incoming branch (unless --no-pull):
   - If `--no-pull` flag is present: Skip this step
   - Otherwise (default behavior):
     - Check if branch exists locally: `git branch --list <branch-name>`
     - If local branch exists:
       ```
       git fetch origin <branch-name>
       git checkout <branch-name>
       git pull origin <branch-name>
       git checkout -  # Return to original branch
       ```
     - If only remote branch exists:
       ```
       git fetch origin <branch-name>:<branch-name>
       ```
     - Show confirmation: "✓ Pulled latest changes from <branch-name>"

### 3. Pre-operation analysis (DETAILED):
   - Show current branch name
   - Show target branch to merge/rebase
   - Show operation mode (MERGE or REBASE)
   - Show commit mode (AUTO_COMMIT or NO_COMMIT)

   **📊 ANALYZE BOTH BRANCHES IN DETAIL:**

   a) **Get commit divergence:**
      - Run `git log --oneline <target>..<current>` to see commits in current branch
      - Run `git log --oneline <current>..<target>` to see commits in target branch
      - Show commit count and list all commit messages from both sides

   b) **Identify changed files in each branch:**
      - Run `git diff --name-status <merge-base>..<current>` for BASE branch changes
      - Run `git diff --name-status <merge-base>..<target>` for INCOMING branch changes
      - Categorize files: Modified (M), Added (A), Deleted (D), Renamed (R)
      - Find files modified in BOTH branches (potential conflicts)

   c) **Analyze the nature of changes:**
      - For each file modified in both branches, show:
        - Number of lines changed in BASE
        - Number of lines changed in INCOMING
        - Brief description of what changed (e.g., "Clock refactoring", "Removed time params")
      - Use `git log --oneline --follow -- <file>` to understand recent changes

   d) **Present a clear summary:**
      ```
      📊 Merge Analysis: <target> → <current>

      BASE Branch (<current>):
        • <commit-count> commits ahead
        • Key changes:
          ✨ <file1>: <description of changes>
          ✨ <file2>: <description of changes>

      INCOMING Branch (<target>):
        • <commit-count> commits
        • Key changes:
          ✨ <file1>: <description of changes>
          ✨ <file2>: <description of changes>

      ⚠️  Files modified in BOTH branches (potential conflicts):
        • <file1>
        • <file2>
      ```

### 4. Attempt operation:
   - If REBASE mode: Run `git rebase <branch-name>`
   - If MERGE mode:
     - If NO_COMMIT mode: Run `git merge <branch-name> --no-commit --no-ff`
     - If AUTO_COMMIT mode: Run `git merge <branch-name>`
   - Capture the result

### 5. If there are conflicts - INTELLIGENT PATTERN ANALYSIS:

   **DO NOT auto-resolve. Instead, analyze ALL conflicts holistically to identify patterns:**

   a) **List all conflicted files** with `git status`

   b) **Read ALL conflicted files** and collect all conflict blocks

   c) **Understand the INTENT of each branch** by analyzing commits:
      - What is BASE trying to achieve? (e.g., "Clock interface refactoring")
      - What is INCOMING trying to achieve? (e.g., "Move time logic to domain")
      - Are these complementary or competing changes?

   d) **Identify PATTERNS across all conflicts** (not file-by-file):

      Group conflicts by type and similarity:

      1. **Function signature changes** (e.g., parameter added/removed)
         - Count how many times this pattern appears
         - List affected functions/files
         - Determine if it's a systematic refactoring

      2. **Implementation differences** (e.g., Clock interface vs direct time.Now())
         - Identify which approach is more complete/mature
         - Check if one side has TODOs or incomplete work

      3. **Documentation/comments**
         - Comments added in both branches
         - Documentation updates

      4. **New files/deletions**
         - Files added in both branches
         - Files deleted in one branch but modified in other

   e) **Analyze completeness and maturity:**
      - Which branch has more complete implementation?
      - Are there TODOs or FIXMEs indicating incomplete work?
      - Does one approach follow established patterns in the codebase?
      - How many places would need changes if we pick one approach?

### 6. Present a SINGLE COMPREHENSIVE RESOLUTION PLAN:

   **CRITICAL:** Present ONE unified plan for ALL conflicts, not file-by-file questions.

   Format the plan like this:

   ```
   📋 CONFLICT RESOLUTION PLAN

   Analyzed <N> conflicts across <M> files. Here's my proposed strategy:

   🎯 OVERALL STRATEGY: [Keep BASE | Keep INCOMING | Hybrid approach]

   📊 REASONING:
   - BASE branch intent: <describe what BASE is doing>
   - INCOMING branch intent: <describe what INCOMING is doing>
   - Maturity assessment: <which is more complete>
   - Architecture alignment: <which follows established patterns>
   - Impact analysis: <what would change with each choice>

   🔧 DETAILED RESOLUTION BY PATTERN:

   Pattern 1: Function signatures - Clock interface vs removed time params
     Occurrences: 29 conflicts across 5 files
     Files: petrinet_workflow.go (6), events_test.go (2), ...

     Proposed: Keep BASE (Clock interface)
     Why:
       - BASE implementation is complete (Clock pattern in 29 places)
       - INCOMING has incomplete refactoring (TODOs present)
       - BASE follows ports & adapters architecture
       - Preserving INCOMING would require reverting entire Clock refactoring

     Action:
       ✓ Keep all Clock interface signatures
       ✓ Preserve TODO comments from INCOMING for future context
       ✓ Result: func createToken(input TokenInput, now time.Time)

   Pattern 2: Documentation updates
     Occurrences: 3 conflicts in markdown files
     Files: ports_session.md (1), README.md (1), next-steps.md (1)

     Proposed: Merge both versions
     Why: Both branches added valuable documentation

     Action:
       ✓ Combine documentation from both branches
       ✓ Preserve structure from BASE, add content from INCOMING

   Pattern 3: New files
     Occurrences: 2 files
     Files: internal/cpn/domain/valueobject/place_id.go, next-steps.md

     Proposed: Keep INCOMING files
     Why: New functionality, no conflicts with BASE

     Action:
       ✓ Accept new files from INCOMING branch

   ⚠️ EXCEPTIONS (if any):
   - [List any conflicts that don't fit the patterns and need special handling]

   🔄 ALTERNATIVE STRATEGIES:

   Alternative A: Keep INCOMING (remove time params)
     Impact:
       • Would require reverting Clock refactoring (29 places)
       • Need to complete domain move (multiple TODOs)
       • Breaks established architecture pattern
       • Risk: High, incomplete refactoring
     Recommendation: Not advised

   Alternative B: Manual merge (hybrid approach)
     Impact:
       • Keep Clock interface but add domain layer
       • Requires additional refactoring work
       • More complex, but potentially better long-term
       • Risk: Medium, requires careful design
     Recommendation: Consider for future iteration

   📊 STATISTICS:
   Total conflicts: 29
   Patterns identified: 3
   Files affected: 5
   Estimated resolution time: ~2 minutes (automated)
   ```

   **Then ask user for approval using AskUserQuestion:**

   ```
   Question: "Does this conflict resolution plan look good?"

   Options:
     1. "Apply this plan"
        Description: "Execute the proposed strategy automatically.
                     All 29 conflicts will be resolved based on the patterns identified."

     2. "Adjust strategy"
        Description: "I want to modify the approach for one or more patterns.
                     Let me specify which patterns need different handling."

     3. "Show more details"
        Description: "Expand specific conflicts or patterns before deciding.
                     Show me the actual code differences."

     4. "Manual resolution"
        Description: "The plan doesn't work for me. I'll resolve conflicts manually in the IDE."
   ```

   If user chooses "Adjust strategy", ask follow-up questions about specific patterns:
   ```
   Question: "Which pattern do you want to adjust?"

   Options:
     1. "Pattern 1: Function signatures (29 conflicts)"
     2. "Pattern 2: Documentation (3 conflicts)"
     3. "Pattern 3: New files (2 conflicts)"
     4. "Multiple patterns"
   ```

   Then for the selected pattern, offer alternatives:
   ```
   Question: "How should we handle Pattern 1 (Function signatures) instead?"

   Options:
     1. "Keep INCOMING (remove time params)"
     2. "Mix: Keep BASE for most, INCOMING for specific files"
     3. "Let me review each conflict individually"
   ```

### 7. Resolve conflicts based on user's choice:

   - If user chose "Keep BASE": Apply ours strategy for those conflicts
   - If user chose "Keep INCOMING": Apply theirs strategy for those conflicts
   - If user chose "Mixed": Ask for each conflict individually
   - If user chose "Manual": Stop and let user resolve in IDE

   **When auto-resolving:**
   - Use Edit tool to resolve conflicts by removing markers
   - Preserve requested comments (like TODOs from INCOMING)
   - Stage resolved files with `git add <file>`
   - Show what was changed in each file

   **For REBASE:** After resolving all conflicts, run `git rebase --continue`

### 8. Verify resolution:
   - Run `git diff --check` to ensure no conflict markers remain
   - Show `git status` to confirm all conflicts are resolved
   - List files that were modified during resolution

### 9. Run tests (MANDATORY):
   - Detect project type and test command:
     - Go projects: `go test ./...`
     - Node.js: `npm test` or `yarn test`
     - Python: `pytest` or `python -m pytest`
     - Java: `mvn test` or `gradle test`

   - Execute the appropriate test command
   - Show test results (passed/failed) with details

   **If tests FAIL:**
     - Show detailed failure output
     - Explain what might have caused the failure based on the merge
     - Ask user using AskUserQuestion:
       ```
       Question: "Tests failed after merge. What would you like to do?"
       Options:
         1. "Abort merge/rebase" - Return to original state
         2. "Show me the failures" - Display full test output for analysis
         3. "Continue anyway" - Complete merge despite test failures (not recommended)
         4. "Let me fix the tests" - Pause for manual fixes then retry
       ```
     - If user chooses abort: `git merge --abort` or `git rebase --abort`

   **If tests PASS:**
     - Confirm all tests passed
     - Show test summary (e.g., "79 tests in 12 packages passed")
     - Proceed to complete the operation

### 10. Complete operation:

   **Check commit mode:**

   - If NO_COMMIT mode (`--no-commit` flag was used):
     ```
     ⚠️  MERGE PAUSED FOR MANUAL REVIEW

     All conflicts have been resolved and tests passed.
     The merge is ready but NOT finalized (--no-commit flag).

     Next steps:
       • Review the changes: git diff --cached
       • Make additional edits if needed
       • Finalize merge: git commit
       • Or abort: git merge --abort

     Files staged for commit: <list files>
     ```
     - **STOP HERE** - Do not create merge commit
     - Let user review and commit manually

   - If AUTO_COMMIT mode (default):
     - For MERGE: Create merge commit with default message
     - For REBASE: Run `git rebase --continue` to complete
     - Show summary of the operation

### 11. Final summary:

   Present a comprehensive summary (skip if NO_COMMIT mode):

   ```
   ✅ MERGE COMPLETED SUCCESSFULLY

   Operation: MERGE feature/OR-1381-add-cpn-worker → OR-1381/remove-time-calls

   📊 Statistics:
     • Commits merged: 4
     • Files changed: 7
     • Conflicts resolved: 11 (across 5 files)

   🔧 Resolution Strategy Used:
     • petrinet_workflow.go (6 conflicts): Kept BASE Clock interface + added TODOs
     • events_test.go (2 conflicts): Kept BASE time.Now() parameters
     • event_dispatcher_test.go (2 conflicts): Kept BASE time.Now() parameters
     • petrinet_workflow_test.go (1 conflict): Kept BASE time.Now() parameter
     • ports_session.md (1 conflict): Merged both documentation updates

   📦 New Files Added:
     • internal/cpn/domain/valueobject/place_id.go
     • next-stpes.md

   ✅ Tests: PASSED (12 packages, 79 tests)

   📝 Next Steps:
     • Review the changes: git diff HEAD~1
     • Push to remote: git push
     • Create PR if needed

   💡 Reasoning:
     Kept Clock interface refactoring from BASE because it's more complete
     and follows the established ports & adapters pattern. Preserved TODO
     comments from INCOMING for future domain refactoring context.
   ```

## Important notes:

- **NEVER auto-resolve without detailed analysis and user confirmation**
- **Always explain WHY a particular resolution is recommended**
- **Present alternatives when they exist**
- **Show the actual code differences, not just file names**
- **Preserve valuable comments/TODOs from both branches when possible**
- **Tests are MANDATORY** - never complete without running tests
- **For REBASE:** Remember "ours" and "theirs" are swapped!
- **If conflicts are too complex:** Recommend manual review instead of forcing auto-resolution
- **DO NOT push changes automatically** - let user review first
- **Auto-pull by default:** Always pull incoming branch first unless `--no-pull` is specified
- **Auto-commit by default:** Finalize merge automatically unless `--no-commit` is specified
- **With --no-commit:** Stop after resolving conflicts and running tests, let user review before committing

## Error handling:

- If branch doesn't exist: Show available branches with `git branch -a`
- If merge/rebase fails with non-conflict errors: Show error and suggest fixes
- If user is in middle of existing merge/rebase: Detect and ask what to do
- If working directory is dirty: Warn and ask user to commit/stash first
