# wt

`wt` is a small shell helper for creating a new Git worktree and copying a
hand-picked set of files from the source repository into the new worktree.

## What it does

1. Creates a new worktree at the requested path.
2. Creates a new branch from the given base branch or commit.
3. Looks for a `.git-worktree-copy` file in the source repository.
4. Copies each listed file or directory into the new worktree.

This is useful for untracked local config files or generated assets you want in
every worktree without committing them.

## Usage

```bash
bin/wt <path> <branch-name> [base-branch]
```

Examples:

```bash
bin/wt ../my-feature feature/api-cleanup
bin/wt ../my-feature feature/api-cleanup main
```

## `.git-worktree-copy`

Inside any repository where you use `wt`, create a file named
`.git-worktree-copy` at the repo root:

```text
# Copy local config into each new worktree
.env.local
config/dev-secrets.json
tmp/fixtures
```

Blank lines and lines starting with `#` are ignored.
