---
name: worktree
description: Create and manage Git worktrees with automatic untracked file synchronization
argument-hint: "<command> [args...]"
allowed-tools: Bash, Read, Write, Glob
---

# Git Worktree Sync

Create Git worktrees and automatically synchronize untracked files (migrations/, .env.local, etc.).

## Available Commands

- `/worktree create <branch> [path]` - Create new worktree and sync files
- `/worktree sync [path]` - Sync files to existing worktree
- `/worktree list` - List all worktrees
- `/worktree clean <path>` - Remove worktree

---

## Execution Instructions

Parse $ARGUMENTS and perform the appropriate action.

### 1. create Command

**Input**: `/worktree create <branch> [path]`

**Steps**:

1. **Verify Git Repository**
   ```bash
   git rev-parse --git-dir
   ```
   - On failure: Output "Current directory is not a Git repository" and exit

2. **Load Configuration Files**
   - Read `.worktree.json` (using Read tool)
   - Read `.worktree.local.json` (if exists)
   - Use defaults if not found:
     ```json
     {
       "includes": [],
       "excludes": [],
       "mode": "symlink",
       "hooks": {},
       "defaultPath": "../worktrees"
     }
     ```
   - Merge rules:
     - `includes`: Union
     - `excludes`: Union
     - `mode`: Local config takes precedence
     - `hooks`: Local config takes precedence
     - `defaultPath`: Local config takes precedence

3. **Determine Worktree Path**
   - Use `[path]` argument if provided
   - Otherwise use `{defaultPath}/{branch}`
   - Example: `../worktrees/feat/new-feature`

4. **Create Worktree**
   ```bash
   git worktree add <path> <branch>
   ```
   - Branch is auto-created if it doesn't exist
   - On failure: Output error and exit

5. **Collect Untracked Files**
   ```bash
   git ls-files --others --exclude-standard
   ```

6. **Filter by Pattern Matching**
   - Using Glob tool or bash:
   - Select only files matching `includes` patterns
   - Exclude files matching `excludes` patterns
   - Example: `includes: ["migrations/", "*.env"]`, `excludes: [".claude/"]`

7. **Sync Files**
   - Find git root path:
     ```bash
     git rev-parse --show-toplevel
     ```
   - For each file:
     - **symlink mode**:
       ```bash
       ln -s <source_absolute_path> <target_absolute_path>
       ```
     - **copy mode**:
       ```bash
       cp -r <source> <target>
       ```
   - Process directories recursively

8. **Execute postCreate Hook** (if defined)
   ```bash
   cd <worktree_path> && <hook_command>
   ```
   - Example: `pnpm install`
   - Continue on failure (show warning only)

9. **Output Result**
   ```
   Worktree created successfully
   Path: <worktree_path>
   Branch: <branch>
   Synced files: <count>
   ```

---

### 2. sync Command

**Input**: `/worktree sync [path]`

**Steps**:

1. **Determine Target Path**
   - Use `[path]` argument if provided
   - Otherwise use current directory

2. **Load Configuration Files** (same as create)

3. **Find Main Worktree Path**
   ```bash
   git worktree list --porcelain | grep "^worktree" | head -1 | cut -d' ' -f2
   ```

4. **Collect Untracked Files from Main**
   ```bash
   cd <main_worktree> && git ls-files --others --exclude-standard
   ```

5. **Filter and Sync** (same as create)

6. **Execute postSync Hook** (if defined)

7. **Output Result**

---

### 3. list Command

**Input**: `/worktree list`

**Steps**:

1. **Get Worktree List**
   ```bash
   git worktree list --porcelain
   ```

2. **Parse and Format Output**
   - Example output:
   ```
   Main Worktree:
     /path/to/main (main) [current]

   Worktrees:
     /path/to/worktree1 (feat/branch-a)
     /path/to/worktree2 (feat/branch-b)
   ```

---

### 4. clean Command

**Input**: `/worktree clean <path>`

**Steps**:

1. **Validate Path**
   - Verify `<path>` is a valid worktree

2. **Remove Worktree**
   ```bash
   git worktree remove <path>
   ```
   - Retry with `--force` on failure

3. **Clean Up Directory** (if remaining)
   ```bash
   rm -rf <path>
   ```

4. **Output Result**

---

## Configuration File Examples

### `.worktree.json` (Project-wide)

```json
{
  "includes": [
    "migrations/",
    "apps/mono/src/scripts/"
  ],
  "excludes": [
    ".claude/",
    ".codex/",
    ".zed/"
  ],
  "mode": "symlink",
  "hooks": {
    "postCreate": "pnpm install"
  },
  "defaultPath": "../worktrees"
}
```

### `.worktree.local.json` (Personal)

```json
{
  "includes": [
    ".env.local",
    ".env.development"
  ],
  "mode": "copy",
  "hooks": {
    "postCreate": "pnpm install && pnpm build:mono"
  }
}
```

---

## Error Handling

- Not a Git repository: Clear error message
- Worktree creation failed: Show git error message
- File sync failed: Show individual file errors but continue
- Hook failed: Show warning and continue

---

## Important Notes

- **symlink mode**: Changes to original files are immediately reflected in all worktrees
- **copy mode**: Completely independent, requires manual sync
- `.worktree.local.json` is gitignored and not committed
