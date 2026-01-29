---
name: worktree
description: Create and manage Git worktrees with automatic untracked file synchronization
argument-hint: "<command> [args...]"
allowed-tools: Bash, Read, Write, Glob
---

# Git Worktree Sync

Create Git worktrees and automatically synchronize untracked files (migrations/, .env.local, etc.).

## Available Commands

- `/worktree create <branch> [path] [from <base>] [and switch]` - Create new worktree and sync files
- `/worktree sync [path]` - Sync files to existing worktree
- `/worktree list` - List all worktrees
- `/worktree clean <path>` - Remove worktree

---

## Execution Instructions

Parse $ARGUMENTS and perform the appropriate action.

### 1. create Command

**Input**: `/worktree create <branch> [path] [from <base>] [and switch]`

**Examples:**
- `/worktree create feat/new` - Create from baseBranch (default: main)
- `/worktree create feat/new from dev` - Create from dev branch
- `/worktree create feat/new from current` - Create from current branch
- `/worktree create feat/new and switch` - Create and switch to worktree
- `/worktree create feat/new from dev and switch` - Create from dev and switch

**Steps**:

1. **Verify Git Repository**
   ```bash
   git rev-parse --git-dir
   ```
   - On failure: Output "Current directory is not a Git repository" and exit

2. **Load Configuration with Priority**
   - Start with defaults:
     ```json
     {
       "includes": [],
       "excludes": [],
       "mode": "symlink",
       "hooks": {},
       "defaultPath": "../worktrees",
       "baseBranch": "main"
     }
     ```
   - Read `.gwts` (team config, if exists)
   - Read `.claude/gwts/worktree.json` (personal config, if exists)
   - Merge rules:
     - `includes`: Union of all configs
     - `excludes`: Union of all configs
     - `mode`: Personal > Team > Default
     - `hooks`: Personal > Team > Default
     - `defaultPath`: Personal > Team > Default
     - `baseBranch`: Personal > Team > Default

3. **Determine Worktree Path**
   - Use `[path]` argument if provided
   - Otherwise use `{defaultPath}/{branch}`
   - Example: `../worktrees/feat/new-feature`

4. **Determine Base Branch**
   - Parse arguments for "from <base>" pattern
   - If "from current": Use current branch
     ```bash
     baseBranch=$(git branch --show-current)
     ```
   - Else if "from <base>": Use specified base
   - Else: Use config's `baseBranch` (default: "main")

5. **Create Worktree**
   - Check if branch exists:
     ```bash
     git rev-parse --verify <branch> 2>/dev/null
     ```
   - If exists: Create worktree from existing branch
     ```bash
     git worktree add <path> <branch>
     ```
   - If not exists: Create new branch from baseBranch
     ```bash
     git worktree add -b <branch> <path> <baseBranch>
     ```
   - On failure: Output error and exit

6. **Collect Untracked Files**
   ```bash
   git ls-files --others --exclude-standard
   ```

7. **Filter by Pattern Matching**
   - Using Glob tool or bash:
   - Select only files matching `includes` patterns
   - Exclude files matching `excludes` patterns
   - Example: `includes: ["migrations/", "*.env"]`, `excludes: [".claude/"]`

8. **Sync Files**
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

9. **Execute postCreate Hook** (if defined)
   ```bash
   cd <worktree_path> && <hook_command>
   ```
   - Example: `pnpm install`
   - Continue on failure (show warning only)

10. **Output Result**
   ```
   Worktree created successfully
   Path: <worktree_path>
   Branch: <branch>
   Synced files: <count>
   ```

11. **Switch to Worktree** (if "and switch" in args)
   ```bash
   cd <worktree_path>
   ```
   - Output: "Switched to: <worktree_path>"
   - Note: This changes the current working directory

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

## Configuration Reference

**Priority:** Personal (`.claude/gwts/worktree.json`) > Team (`.gwts`) > Defaults

**Defaults:**
```json
{
  "includes": [],
  "excludes": [],
  "mode": "symlink",
  "hooks": {},
  "defaultPath": "../worktrees",
  "baseBranch": "main"
}
```

**Options:**

| Option | Type | Description |
|--------|------|-------------|
| `includes` | `string[]` | Patterns to sync (merged across configs) |
| `excludes` | `string[]` | Patterns to skip (merged across configs) |
| `mode` | `"symlink"` \| `"copy"` | symlink=shared, copy=independent |
| `hooks.postCreate` | `string` | Command to run after worktree creation |
| `hooks.postSync` | `string` | Command to run after sync |
| `defaultPath` | `string` | Where to create worktrees |
| `baseBranch` | `string` | Base branch for new branches |

**Pattern syntax:** Standard glob (`migrations/`, `*.config.js`, `apps/*/db/`)

---

## Error Handling

- Not a Git repository → Exit with clear message
- Worktree creation failed → Show git error
- File sync failed → Log error, continue with others
- Hook failed → Show warning, continue
