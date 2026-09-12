---
name: contextit-git
description: >
  claudeit git integration — smart git commands with full project context.
  Trigger when the user types any of these: "claudeit commit", "claudeit push",
  "claudeit pull", "claudeit branch", "claudeit pr", "claudeit log", "claudeit diff",
  "claudeit stash", "claudeit sync", "claudeit status", "claudeit merge",
  "claudeit git <any command>", or asks Claude to commit, push, pull, create a branch,
  write a PR, explain a diff, or do anything git-related inside a claudeit project.
  This skill requires claudeit core (SKILL.md) to be active — it reads project context
  from .contextit/ to make every git operation smarter.
---

# claudeit git — Smart Git with Project Intelligence

claudeit git is not a thin wrapper around git commands. Every operation is informed
by your project context — your conventions, your session decisions, your guardrails.
Claude runs all git commands directly in Claude Code and always confirms before any
write operation.

---

## The Rule: Always Confirm Before Writing

Before ANY write operation — commit, push, merge, stash, branch create — Claude
shows exactly what it will run and waits for confirmation:

```
┌─────────────────────────────────────────────────────────────────┐
│  📝 About to run                                               │
├─────────────────────────────────────────────────────────────────┤
│  git commit -m "feat(checkout): collapse wizard to 2-step"     │
│                                                                 │
│  Confirm? (yes / edit / cancel)                                │
└─────────────────────────────────────────────────────────────────┘
```

- **yes** → runs the command
- **edit** → Claude shows the command for manual editing, then re-confirms
- **cancel** → nothing happens

Destructive commands (force push, reset --hard, clean, rebase) get an extra
warning box in red regardless of this setting.

---

## Command Reference

### Daily Short Commands

| Short form | Equivalent | What claudeit does smarter |
|---|---|---|
| `claudeit status` | `git status` | Plain English summary with session context |
| `claudeit pull` | `git pull` | Pull + summarise what changed vs your work |
| `claudeit push` | `git push` | Pre-push guardrail check → confirm → push |
| `claudeit commit` | `git add -A && git commit` | Generates commit message from session |
| `claudeit branch <name>` | `git checkout -b` | Enforces naming convention from conventions.md |
| `claudeit sync` | pull + merge + push | Safe full sync in one confirmed sequence |
| `claudeit log` | `git log` | Plain English summary of recent commits |
| `claudeit diff` | `git diff` | Diff explained in context of what you're building |
| `claudeit stash` | `git stash push` | Auto-names stash from session context |
| `claudeit pr` | — | Drafts full PR description from session |
| `claudeit merge <branch>` | `git merge` | Merge with conflict explanation if needed |
| `claudeit undo` | `git reset HEAD~1` | Shows what will be undone before running |

### Advanced: `claudeit git <command>`

Passes any git command through with claudeit intelligence:
```
claudeit git rebase -i HEAD~3
claudeit git cherry-pick abc1234
claudeit git bisect start
claudeit git remote -v
claudeit git tag v1.2.0
```

For read-only commands → runs directly.
For write commands → confirms first.
For destructive commands → red warning box + requires typing CONFIRM.

---

## Smart Features

### 1. Commit Message Generation

`claudeit commit` reads the session conversation and generates a commit message
following your conventions from `conventions.md`.

Claude extracts:
- What was built or changed this session
- Which feature it belongs to
- Whether it's a feat, fix, chore, refactor, docs, test

```bash
# Claude runs silently:
git diff --staged
git diff HEAD
```

Then generates:

```
┌─────────────────────────────────────────────────────────────────┐
│  📝 Generated commit message                                   │
├─────────────────────────────────────────────────────────────────┤
│  feat(checkout): collapse 3-step wizard to 2-step              │
│                                                                 │
│  Merged billing and shipping into single step based on         │
│  internal review showing drop-off at Step 2. Progress          │
│  indicator uses shadcn Steps component.                        │
├─────────────────────────────────────────────────────────────────┤
│  Files staged: 4 changed, 87 insertions, 23 deletions          │
│                                                                 │
│  Confirm? (yes / edit / cancel)                                │
└─────────────────────────────────────────────────────────────────┘
```

If no files are staged, Claude asks:
```
Nothing staged. Stage all changes and commit? (yes / select files / cancel)
```

If yes → `git add -A` then commit flow.
If select → Claude lists changed files, user picks which to stage.

---

### 2. PR Description Writer

`claudeit pr` drafts a full pull request description using session context,
the commit log, and the diff.

```bash
git log main..HEAD --oneline
git diff main..HEAD --stat
```

Produces:

```
┌─────────────────────────────────────────────────────────────────┐
│  🔀 Pull Request Draft                                         │
├─────────────────────────────────────────────────────────────────┤
│  Title: feat(checkout): collapse wizard to 2-step flow         │
├─────────────────────────────────────────────────────────────────┤
│  ## What this does                                             │
│  Reduces the checkout flow from 3 steps to 2 by merging        │
│  billing and shipping into a single screen. Internal review    │
│  showed significant drop-off at Step 2.                        │
│                                                                 │
│  ## Changes                                                    │
│  - CheckoutWizard: merged Step 1 + 2                           │
│  - ProgressIndicator: updated to 2-step variant                │
│  - CheckoutForm: combined billing/shipping fields              │
│                                                                 │
│  ## Testing                                                    │
│  - [ ] Checkout flow end-to-end                                │
│  - [ ] Mobile layout at Step 1                                 │
│  - [ ] Back button behaviour                                   │
│                                                                 │
│  ## Guardrails respected                                       │
│  ✅ No modals used                                             │
│  ✅ No third-party analytics added                             │
├─────────────────────────────────────────────────────────────────┤
│  Copy to clipboard? (yes / edit / cancel)                      │
└─────────────────────────────────────────────────────────────────┘
```

Note the **Guardrails respected** section — Claude checks the diff against
`guardrails.md` and confirms nothing was violated.

If the PR title/body should go to GitHub via API in a future version,
this is where that hook lives.

---

### 3. Branch Naming from Conventions

`claudeit branch <description>` reads `conventions.md` and creates a branch
that matches your team format.

Example — conventions.md says `feature/xxx` or `fix/xxx`:

```
claudeit branch "add dark mode toggle"
```

```
┌─────────────────────────────────────────────────────────────────┐
│  🌿 New branch                                                 │
├─────────────────────────────────────────────────────────────────┤
│  git checkout -b feature/dark-mode-toggle                      │
│                                                                 │
│  Confirm? (yes / rename / cancel)                              │
└─────────────────────────────────────────────────────────────────┘
```

If no convention is set, Claude uses `feature/`, `fix/`, `chore/` based on
what the branch is for and asks to confirm before creating.

---

### 4. Pre-Push Guardrail Check

Before every `claudeit push`, Claude runs:

```bash
git diff origin/$(git branch --show-current)..HEAD
```

Then checks the diff against:
- `guardrails.md` — any violations?
- `responsible-ai.md` — if active, any AI principle conflicts?
- `conventions.md` — any obvious convention breaks?

If clean:
```
┌─────────────────────────────────────────────────────────────────┐
│  ✅ Pre-push check passed                                      │
│  No guardrail or convention violations found                   │
├─────────────────────────────────────────────────────────────────┤
│  git push origin feature/dark-mode-toggle                      │
│                                                                 │
│  Confirm? (yes / cancel)                                       │
└─────────────────────────────────────────────────────────────────┘
```

If a violation is found:
```
┌─────────────────────────────────────────────────────────────────┐
│  ⚠️  Pre-push check flagged an issue                           │
├─────────────────────────────────────────────────────────────────┤
│  File:      src/analytics/tracker.js                           │
│  Guardrail: "No third-party analytics"                         │
│  Found:     import MixPanel from 'mixpanel-browser'            │
├─────────────────────────────────────────────────────────────────┤
│  Push anyway? Requires written justification.                  │
│  Or cancel and fix first.                                      │
└─────────────────────────────────────────────────────────────────┘
```

---

### 5. Plain English Git Log

`claudeit log` translates git history into readable summaries.

```bash
git log --oneline -20
git log --stat -5
```

Output:
```
┌─────────────────────────────────────────────────────────────────┐
│  📜 Recent history — Payments Platform                         │
├─────────────────────────────────────────────────────────────────┤
│  Today                                                         │
│  • Collapsed checkout to 2-step (you, 2h ago)                  │
│  • Fixed mobile layout at Step 1 (you, 4h ago)                 │
│                                                                 │
│  Yesterday                                                     │
│  • Stripe webhook integration (you)                            │
│  • Auth flow completed and merged (teammate)                   │
│                                                                 │
│  This week                                                     │
│  • Initial Prisma schema (you)                                 │
│  • Project setup and CI (you)                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

### 6. Diff Explained in Context

`claudeit diff` explains what changed in terms of your project — not just
raw lines added/removed.

```bash
git diff HEAD
git diff --staged
```

Instead of raw diff output:
```
┌─────────────────────────────────────────────────────────────────┐
│  🔍 What changed — in context                                  │
├─────────────────────────────────────────────────────────────────┤
│  CheckoutWizard.tsx                                            │
│    The 3-step wizard is now 2-step. Steps const reduced        │
│    from 3 to 2. BillingForm and ShippingForm are now           │
│    rendered together on Step 1.                                │
│                                                                 │
│  ProgressIndicator.tsx                                         │
│    Updated to receive totalSteps=2. Labels updated:            │
│    "Details & Shipping" and "Payment".                         │
│                                                                 │
│  2 files changed · 87 insertions · 23 deletions               │
└─────────────────────────────────────────────────────────────────┘
```

---

### 7. Smart Stash

`claudeit stash` names the stash from what you're working on.

```
┌─────────────────────────────────────────────────────────────────┐
│  📦 Stash                                                      │
├─────────────────────────────────────────────────────────────────┤
│  git stash push -m "checkout-redesign: mid-step WIP"           │
│                                                                 │
│  Confirm? (yes / rename / cancel)                              │
└─────────────────────────────────────────────────────────────────┘
```

`claudeit stash list` — shows stashes with plain English descriptions.
`claudeit stash pop` — shows what will be restored before running.

---

### 8. Merge Conflict Explanation

When `claudeit merge <branch>` or `claudeit pull` results in conflicts,
Claude reads both versions and explains in project context:

```
┌─────────────────────────────────────────────────────────────────┐
│  ⚠️  Merge conflict — CheckoutWizard.tsx                       │
├─────────────────────────────────────────────────────────────────┤
│  Your version:    3-step wizard (current feature work)         │
│  Incoming:        Original 3-step + billing fix from main      │
│                                                                 │
│  Context: You collapsed this to 2-step this session.           │
│  The incoming change fixes a billing bug in the old 3-step.    │
│                                                                 │
│  Recommendation: Keep your 2-step version and manually         │
│  apply the billing fix logic from the incoming change.         │
│                                                                 │
│  [A] Show incoming billing fix to apply manually               │
│  [B] Open file for manual resolution                           │
│  [C] Accept yours (loses billing fix)                          │
│  [D] Accept incoming (loses 2-step collapse)                   │
└─────────────────────────────────────────────────────────────────┘
```

---

### 9. Safe Sync

`claudeit sync` does a full pull → merge → push safely in one sequence:

```bash
# Claude runs each step, confirms before the next:
git fetch origin
# Shows what's incoming
git pull origin <branch> --rebase
# If conflicts → conflict explanation flow
# If clean:
git push origin <branch>
```

Each step shown and confirmed before proceeding.

---

### 10. Undo

`claudeit undo` always shows what will be undone before doing anything:

```
┌─────────────────────────────────────────────────────────────────┐
│  ↩️  Undo last commit                                          │
├─────────────────────────────────────────────────────────────────┤
│  Will undo: "feat(checkout): collapse wizard to 2-step"        │
│  Committed: 12 minutes ago                                     │
│  Files affected: 4                                             │
│                                                                 │
│  Changes will return to staging area (soft reset).             │
│  Your work is NOT deleted.                                     │
│                                                                 │
│  git reset --soft HEAD~1                                       │
│                                                                 │
│  Confirm? (yes / cancel)                                       │
└─────────────────────────────────────────────────────────────────┘
```

Hard reset is always shown in red with an extra confirmation step.

---

## Destructive Command Protection

Commands that can cause data loss get a red warning box and require
typing `CONFIRM` (not just yes):

```
┌─────────────────────────────────────────────────────────────────┐
│  🔴 DESTRUCTIVE OPERATION                                      │
├─────────────────────────────────────────────────────────────────┤
│  git push --force origin main                                  │
│                                                                 │
│  This will overwrite remote history. Teammates who have        │
│  pulled will have diverged histories. This cannot be undone.   │
├─────────────────────────────────────────────────────────────────┤
│  Type CONFIRM to proceed, or anything else to cancel.          │
└─────────────────────────────────────────────────────────────────┘
```

Destructive commands: `--force`, `--force-with-lease`, `reset --hard`,
`clean -f`, `rebase` (on shared branches), `push` to main/master directly.

---

## Integration with claudeit save

When `claudeit save` runs at the end of a session, it automatically
offers to commit and push the context update:

```
┌─────────────────────────────────────────────────────────────────┐
│  💾 Session context saved                                      │
├─────────────────────────────────────────────────────────────────┤
│  decisions.md  → +1 decision logged                            │
│  features.md   → auth marked shipped                           │
├─────────────────────────────────────────────────────────────────┤
│  Commit and push context to team? (yes / cancel)               │
│                                                                 │
│  git commit -m "context: auth shipped, zustand decision"       │
│  git push origin main                                          │
└─────────────────────────────────────────────────────────────────┘
```

---

## Quick Reference

| Command | What it does |
|---|---|
| `claudeit status` | Status with plain English session summary |
| `claudeit pull` | Pull + what changed summary |
| `claudeit push` | Guardrail check → confirm → push |
| `claudeit commit` | Generate message from session → confirm → commit |
| `claudeit branch <desc>` | Name from conventions → confirm → create |
| `claudeit pr` | Draft PR title + body + guardrail checklist |
| `claudeit log` | Plain English git history |
| `claudeit diff` | Diff explained in project context |
| `claudeit stash` | Auto-named stash → confirm |
| `claudeit stash list` | Stashes with plain English labels |
| `claudeit stash pop` | Show what restores → confirm |
| `claudeit merge <branch>` | Merge with conflict explanation |
| `claudeit sync` | Fetch + pull + push safely, step by step |
| `claudeit undo` | Show what undoes → soft reset → confirm |
| `claudeit git <cmd>` | Any git command with claudeit intelligence |
