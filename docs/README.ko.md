# gwts-cc

> Claude Code를 위한 Git worktree + untracked 파일 동기화

[English](../README.md) | [한국어](README.ko.md)

## 주요 기능

- Worktree 생성 시 untracked 파일 자동 동기화
- Symlink 또는 Copy 모드 선택
- 패턴 기반 필터링 (`includes` / `excludes`)
- 생성 후 Hook 실행 (`pnpm install` 등)
- 프로젝트 설정 + 개인 로컬 설정

## 설치

```bash
/plugin marketplace add BaeSeokJae/gwts-cc
/plugin install gwts-cc@gwts-cc
```

## 빠른 시작

```bash
# Worktree 생성 + 파일 동기화
/worktree create feat/new-feature

# Worktree 목록 확인
/worktree list

# 기존 worktree에 파일 동기화
/worktree sync

# Worktree 제거
/worktree clean ../worktrees/feat/new-feature
```

## 설정

프로젝트 루트에 `.worktree.json` 생성:

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

개인 설정은 `.worktree.local.json`에 (gitignore 포함).

| 옵션 | 설명 | 기본값 |
|------|------|--------|
| `includes` | 동기화할 패턴 | `[]` |
| `excludes` | 제외할 패턴 | `[]` |
| `mode` | `symlink` 또는 `copy` | `symlink` |
| `hooks.postCreate` | 생성 후 실행 | - |
| `hooks.postSync` | 동기화 후 실행 | - |
| `defaultPath` | Worktree 경로 | `../worktrees` |

## 문서

자세한 명령어는 [commands/worktree.md](../commands/worktree.md) 참고.

## 라이선스

MIT
