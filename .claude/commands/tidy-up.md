# Tidy Up

## Goal

Surface stale branches, worktrees, stashes, and abandoned work in the current
repo; summarize findings concisely; propose a cleanup strategy per item; and
execute only after the human has explicitly approved each destructive step.

Use before starting new work, or as a standalone hygiene pass.

## Preconditions

- Running inside a git repository (the skill detects the repo root itself)
- Working tree is readable; no locked index
- Network reachable for `git fetch` (skill degrades gracefully if offline — it
  will flag that ahead/behind counts may be stale)
- Optional: `gh` CLI available for draft-PR inspection and creation

## Steps

### Phase 1 — Detect (read-only)

1. Confirm repo root, current branch, and default branch:

   ```bash
   git rev-parse --show-toplevel
   git symbolic-ref --short HEAD
   git symbolic-ref --short refs/remotes/origin/HEAD 2>/dev/null || echo origin/main
   ```

2. Fetch with pruning so "gone" upstreams and merge status are accurate:

   ```bash
   git fetch --all --prune --tags
   ```

   If this fails (offline, no remote), continue but flag it in the summary.

3. Check for in-progress operations — these trump everything else:

   ```bash
   ls .git/MERGE_HEAD .git/REBASE_HEAD .git/CHERRY_PICK_HEAD \
      .git/BISECT_LOG .git/rebase-merge .git/rebase-apply 2>/dev/null
   ```

4. Inventory the working tree and refs:

   ```bash
   git status --porcelain=v1 --branch
   git stash list
   git worktree list --porcelain
   git for-each-ref --sort=-committerdate refs/heads \
     --format='%(refname:short)%09%(committerdate:iso8601)%09%(upstream)%09%(upstream:track)%09%(contents:subject)'
   ```

5. For each local branch except the default, compute:
   - Last commit date, author, subject
   - Unique commit count vs default:
     `git rev-list --count <default>..<branch>`
   - Merged into default? `git merge-base --is-ancestor <branch> <default>`
   - Has upstream? Ahead/behind? Upstream gone?
     (from `for-each-ref` `%(upstream:track)` or `git branch -vv`)

6. For each non-primary worktree, cd in and run the same status/stash
   inventory. Note the branch checked out there.

7. Look for known WIP detritus at the repo root:
   - `.claude/worktrees/`
   - `tmp/`, `.scratch/`, `wip/`
   - Untracked files flagged by `git status --porcelain`

### Phase 2 — Summarize

Emit a compact, scannable summary. Example shape:

```
Repo: <name>  @ default=<branch>  current=<branch>
Fetch: ok | offline (counts may be stale)
In-progress: none | REBASE | MERGE | CHERRY-PICK | BISECT

Working tree (<current>):
  staged=<n>  modified=<n>  untracked=<n>

Stashes (<n>):
  stash@{0}  "<subject>"  <date>
  ...

Branches (<total>, excluding <default>):
  clean (merged + pushed):   <n>
  unpushed unique work:       <n>
  pushed, ahead of remote:    <n>
  upstream gone:              <n>
  idle >30d:                  <n>

Worktrees (<n>):
  <path>  branch=<b>  status=<clean|dirty>
```

Then a **per-item detail block** for every candidate, each line ≤ 1 screen:

```
[B1] feature/old-auth
     last:   2026-03-02  "wip: refactor login guard"  (53d ago)
     vs main: +7 / -0   merged=no   upstream=gone
     reco:   ISOLATE — push to archive/feature-old-auth-2026-04-24
```

### Phase 3 — Recommend

For every candidate, pick one strategy and cite the evidence:

| Strategy | When | Default action |
|---|---|---|
| **ISOLATE** | Has unique commits, looks like real work, is ambiguous, or upstream is gone | Push to `archive/<name>-YYYY-MM-DD`; optionally open draft PR; then (after re-approval) delete local |
| **CONSOLIDATE** | Subject overlaps with current work; small unique delta | Tag backup, then rebase/cherry-pick onto current branch |
| **PURGE** | Fully merged OR empty (no unique commits) OR clearly abandoned stub | Safe delete only (`-d`, `remove`, `drop`) |
| **DEFER** | Ambiguous, uncertain provenance, or human should decide | No action; flag for discussion |

Bias toward **DEFER** when in doubt. Never recommend PURGE for anything with
unique commits unless it is already merged to default.

### Phase 4 — Present & await approval

1. Show the summary + per-item recommendations in a single message.
2. Ask the human to approve per item or in batches:
   - "approve all PURGE"
   - "approve B1, B3 as ISOLATE"
   - "defer the rest"
3. Do **not** execute any destructive or history-modifying action until you
   have an explicit "yes / approve / go" for that specific item. A general
   "looks good" is not approval for destructive steps — confirm.

### Phase 5 — Execute approved actions

Execute in order from least to most destructive. After each action, show a
one-line confirmation of what changed.

**ISOLATE**

```bash
# 1. Push to a dated archive ref (never force)
git push origin <branch>:refs/heads/archive/<branch>-$(date +%Y-%m-%d)

# 2. Optionally open a draft PR for later review
gh pr create --draft --base <default> --head archive/<branch>-$(date +%Y-%m-%d) \
  --title "archive: <branch>" --body "Isolated by /tidy-up on $(date +%Y-%m-%d)"

# 3. Only after push succeeds AND human re-approves:
git branch -d <branch>   # safe delete; escalate to -D only with explicit re-approval
```

**CONSOLIDATE**

```bash
# Always create a safety tag first
git tag backup/<branch>-$(date +%Y-%m-%d) <branch>

# Then rebase or cherry-pick onto the current branch
git rebase --onto <current> <default> <branch>
# or
git cherry-pick <sha>..<sha>
```

Leave the original branch and backup tag in place until the human confirms
the result.

**PURGE**

```bash
# Branches — safe delete only
git branch -d <branch>

# Stashes — one at a time
git stash drop stash@{<N>}

# Worktrees — never --force without explicit approval
git worktree remove <path>
```

**After every action**: re-run `git status` + `git branch -vv` and show a
short diff of state so the human can verify.

## Stop conditions

- Every candidate has been classified and either executed or explicitly
  deferred by the human
- The repo is on the same branch it started on, with the same or a cleaner
  working tree
- A final summary lists: archive branches pushed, backup tags created,
  branches/stashes/worktrees removed, and anything deferred
- Nothing was pushed, force-pushed, rebased, or deleted without per-item
  human approval

## Guardrails

- **Never** run any of these without explicit, per-item human approval:
  - `git branch -D` (force delete)
  - `git push --force` or `--force-with-lease`
  - `git reset --hard`
  - `git clean -fd` / `-fdx`
  - `git stash drop` / `git stash clear`
  - `git worktree remove --force`
  - `rm -rf` anywhere inside the repo
- **Never** rewrite history on branches that exist on a remote
- **Never** touch the default branch (main/master) destructively
- **Never** operate on the branch the human is currently working on without
  asking first
- Default to **DEFER** when classification is uncertain
- Always create a `backup/<branch>-<date>` tag before CONSOLIDATE
- Always prefer `-d` over `-D`, `remove` over `remove --force`, `drop` one at
  a time over `clear`
- If `git fetch` failed, say so in the summary — do not silently rely on
  stale merge/ahead/behind data
- If any command fails unexpectedly, stop and surface it to the human before
  continuing

## Project context — control-plane repo

This repo runs on `main` only (see `CLAUDE.md`). In this repo specifically:

- There should be **no** local branches other than `main` and **no**
  worktrees. If any are found, default to **ISOLATE** (if they contain real
  work) or **PURGE** (if empty / merged).
- `.claude/worktrees/` entries should not exist — recommend PURGE.
- Uncommitted coordination changes on `main` (PBIs, handoffs, repo indexes)
  should be committed promptly in small logical units rather than stashed
  or left dangling. Recommend "commit now" rather than "stash".
- Do not recommend CONSOLIDATE here — control-plane work belongs on `main`
  directly, and feature work belongs in its own product repo.
