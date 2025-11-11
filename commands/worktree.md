---
description: Create a git worktree with environment setup (copies .idea, symlinks CLAUDE.md, handles unversioned files)
---

Create a new git worktree with the following steps:

1. **Ask for parameters:**
   - Worktree directory name (required)
   - Branch or commit to checkout (optional, defaults to current branch)

2. **Create the worktree:**
   - Run `git worktree add <directory> [branch/commit]`
   - The directory path should be relative to the current git repository root

3. **Setup environment:**

   a. **Copy .idea directory:**
      - If `.idea/` exists in the current worktree, copy it recursively to the new worktree
      - Use `cp -r .idea <new-worktree>/.idea`

   b. **Symlink CLAUDE.md (if needed):**
      - Check if the original worktree has a `CLAUDE.md` file (not symlink)
      - If it does, create a symlink in the new worktree pointing to it: `ln -s <relative-path-to-original-CLAUDE.md> <new-worktree>/CLAUDE.md`
      - If the original doesn't have `CLAUDE.md`, skip this step (the new worktree will inherit from parent directories automatically)

   c. **Handle other unversioned files:**
      - List unversioned files in current worktree (excluding `.idea` and `CLAUDE.md`)
      - Use `git ls-files --others --exclude-standard` to find them
      - For each unversioned file/directory found, ask the user:
        - "Found unversioned file/directory: `<path>`. Do you want to:"
        - Options: "Copy", "Symlink", "Skip"
      - Execute the chosen action for each file

4. **Confirm completion:**
   - Report the new worktree location
   - List what was copied/symlinked
   - Show the branch/commit checked out

**Important notes:**
- Always use relative paths for symlinks when possible
- Handle errors gracefully (e.g., worktree already exists, branch doesn't exist)
- Don't proceed if the target directory already exists
- Preserve file permissions when copying
