# gwts-cc

> Git worktree + untracked file sync for Claude Code

[English](README.md) | [한국어](docs/README.ko.md)

## Why?

Git worktrees are great for parallel development, but untracked files (migrations, .env, scripts) don't come along. This plugin syncs them automatically.

## Installation

```bash
/plugin marketplace add BaeSeokJae/gwts-cc
/plugin install gwts-cc@gwts-cc
```

## Usage

```bash
/worktree create feat/my-feature    # Create worktree + sync files
/worktree list                      # List worktrees
/worktree sync                      # Sync files to current worktree
/worktree clean <path>              # Remove worktree
```

## Configuration

Create `.gwts` in your project root:

```json
{
  "includes": ["migrations/", ".env.example"],
  "mode": "symlink",
  "hooks": { "postCreate": "pnpm install" }
}
```

That's it. Personal overrides go in `.claude/gwts/worktree.json`.

### Options

| Option | Description | Default |
|--------|-------------|---------|
| `includes` | Files/dirs to sync | `[]` |
| `excludes` | Files/dirs to skip | `[]` |
| `mode` | `symlink` or `copy` | `symlink` |
| `hooks.postCreate` | Command after create | - |
| `defaultPath` | Worktree location | `../worktrees` |

### Mode

| Mode | Behavior | Use for |
|------|----------|---------|
| `symlink` | Links to original, changes sync instantly | migrations, shared configs |
| `copy` | Independent copy, no sync | .env.local, credentials |

### Patterns

```json
{
  "includes": [
    "migrations/",          // directory
    ".env.example",         // file
    "apps/*/db/"            // glob
  ]
}
```

## Examples

**Node.js project:**
```json
{
  "includes": ["migrations/", ".env.example"],
  "hooks": { "postCreate": "npm install" }
}
```

**Monorepo:**
```json
{
  "includes": ["packages/*/migrations/", "apps/*/db/"],
  "hooks": { "postCreate": "pnpm install" }
}
```

**Personal config** (`.claude/gwts/worktree.json`):
```json
{
  "includes": [".env.local"],
  "mode": "copy"
}
```

## License

MIT
