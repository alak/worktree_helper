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
bin/wt -s <branch-name>
bin/wt --short <branch-name>
bin/wt -p [base-branch] [--dry-run] [--force-dirty]
bin/wt --prune [base-branch] [--dry-run] [--force-dirty]
```

Examples:

```bash
bin/wt ../my-feature feature/api-cleanup
bin/wt ../my-feature feature/api-cleanup main
bin/wt -s feature/api-cleanup
bin/wt --prune
bin/wt --prune main
bin/wt --prune main --dry-run
bin/wt --prune main --force-dirty
```

Short mode is a convenience form for creating feature worktrees from `main`.
This command:

```bash
bin/wt -s aaa/bbb
```

expands to:

```bash
bin/wt ../aaa/bbb aaa/bbb main
```

After creating the worktree, `wt` enters the new directory when you run it in
an interactive terminal. If you call it from a non-interactive script, it
prints the `cd` command instead.

## Cleanup Merged Worktrees

Use prune mode to remove linked worktrees whose branches have already been
merged into a base branch:

```bash
bin/wt --prune
bin/wt --prune main
bin/wt --prune main --dry-run
bin/wt --prune main --force-dirty
```

By default, prune mode uses `main` as the merge target. For each linked
worktree, `wt` will:

- skip the main repository worktree
- skip the current worktree and current branch
- skip dirty worktrees with uncommitted changes
- skip branches that are neither ancestry-merged nor patch-equivalent to the base branch
- remove the worktree for merged branches
- delete the merged local branch
- run `git worktree prune` at the end

This makes it easy to clean up feature worktrees after they have landed.

Patch-equivalent branches are branches whose changes already landed in the base
branch through squash, cherry-pick, or rebase-style history, even if the exact
branch commit is not an ancestor of the base branch.

Use `--dry-run` to preview what would be removed without changing anything:

```bash
bin/wt --prune main --dry-run
```

Use `--force-dirty` to also remove merged worktrees that still have uncommitted
changes:

```bash
bin/wt --prune main --force-dirty
```

This is intentionally sharp: it allows `git worktree remove --force`, so local
uncommitted changes in those merged worktrees will be discarded.

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

Each non-comment line should be a path relative to the repository root. The
path can point to either:

- a file, such as `.env.local`
- a directory, such as `tmp/fixtures`

When you run `wt`, the script first creates the new worktree with `git worktree
add`, then it reads `.git-worktree-copy` from the original repository and
copies every listed path into the new worktree at the same relative location.

This is mainly helpful for local-only files that are not committed to Git but
that you still want available in every new worktree, for example:

- local environment files
- development secrets
- generated fixtures
- machine-specific config

Example workflow:

```bash
cd my-repo
printf ".env.local\nconfig/dev-secrets.json\n" > .git-worktree-copy
wt ../my-repo-feature feature/api-cleanup main
```

If `my-repo/.env.local` exists, it will be copied to
`../my-repo-feature/.env.local`. If a listed path does not exist, `wt` skips it
and prints a message.
