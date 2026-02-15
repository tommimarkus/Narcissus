# Git Workflow (Task Start / Task Complete)

This workflow is optimized for short AI-assisted tasks so progress is never lost.

## One-Time Setup
Enable tracked repository hooks:

```bash
git config core.hooksPath .githooks
```

Verify:

```bash
git config --get core.hooksPath
```

Expected output: `.githooks`

## Safety Rule
- Every `git rebase` triggers `.githooks/pre-rebase`.
- The hook creates a backup branch:
  - `backup/<current-branch>/<YYYYMMDD-HHMMSS>`
- If a rebase goes wrong, recover from the backup branch.

## Task Start Cycle
0. Branch guard before starting a new task:

```bash
git branch --show-current
```

- If current branch is `main`: start normally.
- If current branch is not `main`:
  - If current branch work is clearly complete, finish it locally first (merge/cherry-pick as needed), then switch to `main`.
  - If uncertain whether it is complete, ask the user before starting the new task.

1. Sync main:

```bash
git fetch origin
git checkout main
git pull --rebase origin main
```

2. Create a task branch:

```bash
git checkout -b <type>/<YYYY-MM-DD>-<slug>
```

Examples:
- `feat/2026-02-14-settings-toggle`
- `fix/2026-02-14-minimap-tooltip`

3. Optional early checkpoint if you already changed files:

```bash
git add -A
git commit -m "wip: task start <slug>"
```

## During Task
- Create checkpoint commits every 20-45 minutes:

```bash
git add -A
git commit -m "wip: <what changed>"
```

- Before push/finalization, keep branch current:

```bash
git fetch origin
git rebase origin/main
```

## Task Complete Cycle
1. Confirm clean intent:

```bash
git status -sb
```

2. If needed, clean up WIP history:

```bash
git rebase -i origin/main
```

3. Final commit (if additional staged changes remain):

```bash
git add -A
git commit -m "<feat|fix|refactor|chore>: <final summary>"
```

4. Choose completion mode:

Local complete (default while iterating):
- Keep the branch local and do not push.
- Switch back to `main` for the next task.

```bash
git checkout main
```

Publish complete (when ready to share/review):

```bash
git push -u origin HEAD
```

5. If published, open PR to `main`.

## Local Branch Hygiene
When a local task branch is done and merged/cherry-picked later, delete it locally:

```bash
git branch -d <task-branch>
```

If Git blocks deletion because it is unmerged and you intentionally want to remove it:

```bash
git branch -D <task-branch>
```

## Completion Proof (Required)
At the end of every task, capture and share these exact commands/output summary:

```bash
git branch --show-current
git log --oneline -n 3
git status -sb
```

And explicitly state one of:
- `Pushed`: commit is on remote (`origin/<branch>`).
- `Local only`: commit exists locally but has not been pushed yet.

## Recovery
List backup branches:

```bash
git branch --list "backup/*"
```

Recover by checking out a backup branch and either:
- cherry-picking needed commits, or
- resetting your task branch to that backup point.
