# gwts-cc

> Claude Code용 Git worktree + untracked 파일 동기화

[English](../README.md) | [한국어](README.ko.md)

## 왜?

Git worktree는 병렬 개발에 좋지만, untracked 파일(migrations, .env, scripts)은 따라오지 않습니다. 이 플러그인이 자동으로 동기화합니다.

## 설치

```bash
/plugin marketplace add BaeSeokJae/gwts-cc
/plugin install gwts-cc@gwts-cc
```

## 사용법

```bash
/worktree create feat/my-feature    # Worktree 생성 + 파일 동기화
/worktree list                      # Worktree 목록
/worktree sync                      # 현재 worktree에 파일 동기화
/worktree clean <path>              # Worktree 삭제
```

## 설정

프로젝트 루트에 `.gwts` 생성:

```json
{
  "includes": ["migrations/", ".env.example"],
  "mode": "symlink",
  "hooks": { "postCreate": "pnpm install" }
}
```

끝. 개인 설정은 `.claude/gwts/worktree.json`에.

### 옵션

| 옵션 | 설명 | 기본값 |
|------|------|--------|
| `includes` | 동기화할 파일/디렉토리 | `[]` |
| `excludes` | 제외할 파일/디렉토리 | `[]` |
| `mode` | `symlink` 또는 `copy` | `symlink` |
| `hooks.postCreate` | 생성 후 실행할 명령 | - |
| `defaultPath` | Worktree 위치 | `../worktrees` |

### 모드

| 모드 | 동작 | 용도 |
|------|------|------|
| `symlink` | 원본 링크, 변경 즉시 반영 | migrations, 공유 설정 |
| `copy` | 독립 복사본, 동기화 안됨 | .env.local, 인증정보 |

### 패턴

```json
{
  "includes": [
    "migrations/",          // 디렉토리
    ".env.example",         // 파일
    "apps/*/db/"            // glob
  ]
}
```

## 예시

**Node.js 프로젝트:**
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

**개인 설정** (`.claude/gwts/worktree.json`):
```json
{
  "includes": [".env.local"],
  "mode": "copy"
}
```

## 라이선스

MIT
