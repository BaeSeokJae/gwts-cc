# gwts-cc

> Git worktrees with untracked file sync for Claude Code

[English](README.md) | [한국어](docs/README.ko.md)

## Features

- Automatic untracked file sync when creating worktrees
- Symlink or copy mode
- Pattern-based filtering (`includes` / `excludes`)
- Post-create hooks (`pnpm install`, etc.)
- Project config + personal local config

## Installation

```bash
/plugin marketplace add BaeSeokJae/gwts-cc
/plugin install gwts-cc@gwts-cc
```

## Quick Start

```bash
# Create worktree with synced files
/worktree create feat/new-feature

# List all worktrees
/worktree list

# Sync files to existing worktree
/worktree sync

# Remove worktree
/worktree clean ../worktrees/feat/new-feature
```

## Configuration

Create `.worktree.json` in your project root:

```json
{
  "includes": ["migrations/", ".env.local"],
  "excludes": [".claude/"],
  "mode": "symlink",
  "hooks": {
    "postCreate": "pnpm install"
  },
  "defaultPath": "../worktrees"
}
```

Personal overrides go in `.worktree.local.json` (gitignored).

| Option | Description | Default |
|--------|-------------|---------|
| `includes` | Patterns to sync | `[]` |
| `excludes` | Patterns to exclude | `[]` |
| `mode` | `symlink` or `copy` | `symlink` |
| `hooks.postCreate` | Run after create | - |
| `hooks.postSync` | Run after sync | - |
| `defaultPath` | Worktree location | `../worktrees` |

## Documentation

See [commands/worktree.md](commands/worktree.md) for detailed command reference.

## License

MIT
