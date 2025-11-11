---
description: Merge or rebase another branch into the current branch with intelligent conflict analysis and resolution
tags: [project, gitignored]
---

You are tasked with merging or rebasing another branch into the current branch with intelligent conflict analysis and interactive resolution.

## Usage:
- `/merge main` - merge main into current branch
- `/merge main -r` - rebase current branch onto main
- `/merge -r main` - rebase current branch onto main
- `/merge main --rebase` - rebase current branch onto main

## Steps to follow:

### 1. Validate input:
   - Parse arguments to detect branch name and flags
   - Check for `-r` or `--rebase` flag in arguments
   - Ensure a branch name was provided
   - Verify the branch exists locally or remotely
   - Set operation mode: MERGE or REBASE

### 2. Pre-operation analysis (DETAILED):
   - Show current branch name
   - Show target branch to merge/rebase
   - Show operation mode (MERGE or REBASE)

   **�� ANALYZE BOTH BRANCHES IN DETAIL:**

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
      �� Merge Analysis: <target> → <current>

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

### 3. Attempt operation:
   - If REBASE mode: Run `git rebase <branch-name>`
   - If MERGE mode: Run `git merge <branch-name>`
   - Capture the result

### 4. If there are conflicts - INTELLIGENT ANALYSIS:

   **DO NOT auto-resolve. Instead, analyze each conflict in detail:**

   a) **List all conflicted files** with `git status`

   b) **For EACH conflicted file, provide detailed analysis:**

      Read the file and identify all conflict blocks (<<<<<<< HEAD markers)

      For each conflict block:
      1. **Show the conflict context** (function/class name, line numbers)
      2. **Explain what BASE changed** (what you did in current branch)
      3. **Explain what INCOMING changed** (what the other branch did)
      4. **Identify the conflict type:**
         - Function signature change (parameters added/removed)
         - Logic/implementation difference
         - Variable/field name change
         - Comment/TODO addition
         - Import/dependency change
         - Formatting/style difference

      5. **Propose a specific resolution strategy with reasoning:**
         ```
         ⚠️  CONFLICT #1: petrinet_workflow.go:249-253

         Function: createTokenFromInput()

         BASE (current branch):
           func createTokenFromInput(input TokenInput, now time.Time) *Token
           - Uses Clock interface: w.clock.Now(ctx)
           - Part of ports & adapters refactoring
           - Already integrated in 29 places

         INCOMING (feature branch):
           func createTokenFromInput(input TokenInput) *Token
           - Removed time parameter
           - TODO comment: "Move to Domain"
           - Incomplete refactoring

         �� PROPOSED RESOLUTION:
           Strategy: Keep BASE (Clock interface approach)

           Reasoning:
           - BASE is more complete (Clock pattern fully implemented)
           - BASE follows ports & adapters architecture
           - INCOMING change is partial/incomplete
           - Can preserve INCOMING's TODO comment for context

           Result will be:
           // TODO: Move to Domain, Constructor responsibility
           func createTokenFromInput(input TokenInput, now time.Time) *Token {
               // ... uses now parameter
           }
         ```

      6. **Alternative strategies (if applicable):**
         ```
         �� ALTERNATIVES:
           A. Keep INCOMING - Would require:
              • Reverting Clock refactoring (29 changes)
              • Completing the domain move (TODO)
              • Risk: Breaks existing pattern

           B. Manual merge - Combine both:
              • Keep Clock interface
              • Add domain responsibility layer
              • Requires: Additional refactoring
         ```

   c) **After analyzing ALL conflicts, present summary:**
      ```
      �� CONFLICT RESOLUTION SUMMARY

      Found <N> conflicts in <M> files:

      File: petrinet_workflow.go (6 conflicts)
        ✓ Conflict #1 (line 249): Keep BASE + add TODO
        ✓ Conflict #2 (line 1264): Keep BASE signature
        ✓ Conflict #3 (line 1345): Keep BASE context param
        ... etc

      File: events_test.go (2 conflicts)
        ✓ Conflict #1 (line 31): Keep BASE time.Now()
        ✓ Conflict #2 (line 50): Keep BASE time.Now()

      File: ports_session.md (1 conflict)
        ✓ Conflict #1 (line 728): Merge both documentation updates
      ```

### 5. Ask for confirmation with detailed options:

   **IMPORTANT:** Use AskUserQuestion tool to present ALL conflicts with options.

   For each conflict category (or file with multiple similar conflicts), ask:

   ```
   Question: "How should we resolve conflicts in petrinet_workflow.go?
             BASE adds Clock interface (w.clock.Now), INCOMING removes time params."

   Options:
     1. "Keep ALL from BASE (Clock interface)"
        Description: "Maintain Clock refactoring for all 6 conflicts.
                     Preserves ports & adapters pattern. Add TODO comments from INCOMING."

     2. "Keep ALL from INCOMING (remove time params)"
        Description: "Use feature branch approach for all 6 conflicts.
                     Will need to revert Clock refactoring across codebase."

     3. "Mixed strategy - decide per conflict"
        Description: "Review each of the 6 conflicts individually and
                     choose resolution case by case."

     4. "Manual resolution"
        Description: "Show me the conflicts and I'll resolve them manually in the IDE."
   ```

   **For documentation/comment conflicts:**
   ```
   Question: "How to resolve documentation conflict in ports_session.md?"

   Options:
     1. "Keep BASE version"
     2. "Keep INCOMING version"
     3. "Merge both (combine content)"
     4. "Manual edit"
   ```

### 6. Resolve conflicts based on user's choice:

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

### 7. Verify resolution:
   - Run `git diff --check` to ensure no conflict markers remain
   - Show `git status` to confirm all conflicts are resolved
   - List files that were modified during resolution

### 8. Run tests (MANDATORY):
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

### 9. Complete operation:
   - For MERGE: Create merge commit (if needed)
   - For REBASE: Ensure rebase completes successfully
   - Show summary of the operation

### 10. Final summary:

   Present a comprehensive summary:

   ```
   ✅ MERGE COMPLETED SUCCESSFULLY

   Operation: MERGE feature/OR-1381-add-cpn-worker → OR-1381/remove-time-calls

   �� Statistics:
     • Commits merged: 4
     • Files changed: 7
     • Conflicts resolved: 11 (across 5 files)

   �� Resolution Strategy Used:
     • petrinet_workflow.go (6 conflicts): Kept BASE Clock interface + added TODOs
     • events_test.go (2 conflicts): Kept BASE time.Now() parameters
     • event_dispatcher_test.go (2 conflicts): Kept BASE time.Now() parameters
     • petrinet_workflow_test.go (1 conflict): Kept BASE time.Now() parameter
     • ports_session.md (1 conflict): Merged both documentation updates

   �� New Files Added:
     • internal/cpn/domain/valueobject/place_id.go
     • next-stpes.md

   ✅ Tests: PASSED (12 packages, 79 tests)

   �� Next Steps:
     • Review the changes: git diff HEAD~1
     • Push to remote: git push
     • Create PR if needed

   �� Reasoning:
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

## Error handling:

- If branch doesn't exist: Show available branches with `git branch -a`
- If merge/rebase fails with non-conflict errors: Show error and suggest fixes
- If user is in middle of existing merge/rebase: Detect and ask what to do
- If working directory is dirty: Warn and ask user to commit/stash first
