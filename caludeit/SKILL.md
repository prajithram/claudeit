---
name: contextit
description: >
  contextit gives Claude persistent project memory across sessions. It extends Claude's
  built-in CLAUDE.md system with structured SDLC context — tech stack, architecture,
  UI theme, product features, major decisions, team conventions, and guardrails.
  Trigger ONLY when the user types "claudeit" followed by a command or alone.
  Valid triggers: "claudeit", "claudeit init", "claudeit load", "claudeit save",
  "claudeit update", "claudeit status", "claudeit join", "claudeit login".
  Do NOT trigger on any other phrase or keyword.
---

# contextit — Persistent Project Memory for Claude Code

## Trigger

This skill activates **only** when the user types `claudeit` (alone or with a subcommand).
No other phrase triggers it.

```
claudeit          ← entry point — always starts here
claudeit init     ← first-time setup (asks SOLO or distributed)
claudeit load     ← start of session
claudeit save     ← end of session
claudeit update   ← mid-session context update
claudeit status   ← project snapshot
claudeit join     ← join a distributed project
claudeit login    ← set up git auth
```

---

## Entry Point — `claudeit`

When the user types `claudeit` with no subcommand, or `claudeit init` for the first time,
Claude checks whether a `.contextit/` folder already exists in the project root:

```bash
ls .contextit/.contextit.json 2>/dev/null
```

**If found** → project already initialized. Run `claudeit load` automatically.

**If not found** → first-time setup. Claude asks ONE question before anything else:

---

### First-Time Mode Selection

Claude presents this exactly:

```
👋 Welcome to claudeit — project memory for Claude Code.

Before we set up, I need to know how you're working on this project:

  [1] SOLO — Just you. No team, no git required.
          Context is saved locally on this machine.
          Perfect for personal projects, freelance work, or solo exploration.

  [2] DISTRIBUTED — A team of developers sharing the same project.
          Context is stored in a shared git repo.
          Every developer runs `claudeit load` to get the latest team context.
          Decisions made by one developer become context for everyone.

Which fits your project? (1 or 2)
```

Claude waits for the answer. No other questions asked yet.

**If 1 → branch to SOLO init flow**
**If 2 → branch to DISTRIBUTED init flow**

---

## SOLO Flow

### `claudeit init` (SOLO)

After mode selection Claude says:
```
Got it — SOLO mode. Let me scan your project and set up context.
This is a one-time setup. After this, `claudeit load` at the start of
every session is all you need.
```

Then runs in sequence:

**Step 1 — Write permissions (always first)**
```bash
mkdir -p .contextit
cat > .contextit/settings.json << 'EOF'
{
  "permissions": {
    "allow": ["Write(.contextit/**)", "Edit(.contextit/**)", "Write(CLAUDE.md)"]
  }
}
EOF
```
Note: this single write may prompt once — it's the last prompt you'll ever see.

**Step 2 — Scan codebase silently**
```bash
ls -la
cat package.json 2>/dev/null || cat requirements.txt 2>/dev/null || cat Cargo.toml 2>/dev/null
cat README.md 2>/dev/null | head -60
cat CLAUDE.md 2>/dev/null
```

**Step 3 — Interview** (see `references/init-interview.md`)

CRITICAL RULE: One question at a time. Ask. Wait for answer. Ask the next.
Never group questions. Never list questions. Never say "I need a few details"
followed by multiple questions. Each topic is its own turn.

Claude asks only about what it could not detect from the scan:
stack gaps → UI/theme → features → decisions → conventions → guardrails (one per turn)

**Step 4 — Write `.contextit/` files**
```bash
# Writes: stack.md, architecture.md, ui.md, features.md,
#         decisions.md, conventions.md, guardrails.md, .contextit.json
```

**Step 5 — Extend CLAUDE.md**
```bash
echo -e "\n## Project Context\nFull SDLC context in \`.contextit/\`. Run \`claudeit load\` at session start." >> CLAUDE.md
```

**Step 6 — Confirm**
```
✅ claudeit initialized (SOLO)

I now know your project:
  Stack:       [detected]
  UI:          [detected]
  Features:    [N captured]
  Decisions:   [N logged]
  Conventions: [detected or TODO]
  Guardrails:  [N set]

Run `claudeit load` at the start of every future session.
Run `claudeit save` at the end of every session to keep context current.
```

---

### `claudeit load` (SOLO)

Reads all `.contextit/` files and briefs Claude for the session:

```bash
for f in stack architecture ui features decisions conventions guardrails; do
  cat .contextit/$f.md 2>/dev/null
done
cat .contextit/.contextit.json
```

Claude delivers a session briefing:
```
📦 claudeit loaded — [Project Name] (SOLO)

Stack:         [stack summary]
UI:            [theme, component lib, colors]
Now building:  [current in-progress feature]
Last decision: [most recent decisions.md entry]
Guardrails:    [top active guardrails]

What are we working on today?
```

Claude is now fully context-aware. No re-explaining needed.

---

### `claudeit save` (SOLO)

End of session. Claude reviews the conversation, extracts changes, shows a diff:

```
📝 Saving session context:

decisions.md  → +1: "Chose Zustand over Redux — simpler for this scale"
features.md   → auth: building → shipped
               → dark mode: added to planned

Save? [yes / edit first]
```

After confirmation Claude writes directly to `.contextit/` files.

---

## DISTRIBUTED Flow

The distributed flow has two roles:
- **Project Owner** — runs `claudeit init` once to create and push the shared context
- **Team Member** — runs `claudeit join <url>` to connect and load context

Claude walks each person through their path step by step, one instruction at a time.

---

### `claudeit init` (DISTRIBUTED — Project Owner)

After the user selects DISTRIBUTED, Claude guides them through four phases.
Each phase is explained to the user before Claude does anything.

---

#### PHASE 1 — Create the context repo on GitHub/GitLab

Claude says:
```
┌─────────────────────────────────────────────────────────────────┐
│  DISTRIBUTED MODE — Phase 1 of 4: Create your context repo     │
├─────────────────────────────────────────────────────────────────┤
│  First, create an empty repo on GitHub or GitLab.              │
│  This is where your team's shared project context will live.   │
│                                                                 │
│  GitHub:                                                        │
│    1. Go to github.com → click "+" → "New repository"          │
│    2. Name it something like: my-project-context               │
│    3. Set visibility: Private (recommended) or Public           │
│    4. ⚠️  Do NOT initialise with README, .gitignore or licence  │
│    5. Click "Create repository"                                 │
│    6. Copy the HTTPS URL shown — looks like:                   │
│       https://github.com/your-org/my-project-context.git       │
│                                                                 │
│  GitLab:                                                        │
│    1. Go to gitlab.com → "New project" → "Create blank project"│
│    2. Name it, set visibility, uncheck "initialise repository" │
│    3. Copy the HTTPS clone URL                                  │
│                                                                 │
│  Paste your repo URL here when ready.                           │
└─────────────────────────────────────────────────────────────────┘
```

Claude waits for the URL. Validates it looks like a proper HTTPS git URL.
If SSH format is pasted (`git@github.com:...`) Claude says:
```
⚠️  That looks like an SSH URL. claudeit uses HTTPS only.
    Use the HTTPS URL instead — it starts with https://github.com/...
    You can find it on your repo page under "Code" → "HTTPS".
```

---

#### PHASE 2 — Set up authentication (PAT)

Claude says:
```
┌─────────────────────────────────────────────────────────────────┐
│  DISTRIBUTED MODE — Phase 2 of 4: Authenticate with git        │
├─────────────────────────────────────────────────────────────────┤
│  I need a Personal Access Token to push context to your repo.  │
│  This is stored securely in your OS keychain — never in any    │
│  file or committed to git.                                      │
│                                                                 │
│  Create your token:                                             │
│                                                                 │
│  GitHub:                                                        │
│    1. github.com → profile photo → Settings                    │
│    2. Left sidebar → Developer settings (at the bottom)        │
│    3. Personal access tokens → Tokens (classic)                │
│    4. Generate new token (classic)                              │
│    5. Note: claudeit                                            │
│    6. Expiration: 90 days or "No expiration"                   │
│    7. Scopes: ✅ repo (tick the top-level repo checkbox)       │
│    8. Generate token → COPY IT NOW (shown only once)           │
│                                                                 │
│  GitLab:                                                        │
│    1. gitlab.com → avatar → Preferences                        │
│    2. Access Tokens → Add new token                            │
│    3. Name: claudeit                                            │
│    4. Scopes: ✅ read_repository  ✅ write_repository          │
│    5. Create → COPY IT NOW (shown only once)                   │
│                                                                 │
│  Paste your token here when ready.                              │
└─────────────────────────────────────────────────────────────────┘
```

Claude waits for the token. Then:

```bash
# 1 — detect OS and configure credential helper
uname -s   # Darwin=macOS, Linux, check for WSL

# macOS
git config --global credential.helper osxkeychain

# Linux — try libsecret first, fallback to store
git-credential-libsecret --help > /dev/null 2>&1   && git config --global credential.helper      /usr/lib/git-core/git-credential-libsecret   || git config --global credential.helper store

# Windows WSL
git config --global credential.helper   "/mnt/c/Program Files/Git/mingw64/bin/git-credential-manager.exe"   2>/dev/null || git config --global credential.helper store

# 2 — store token in OS keychain (never written to any file)
printf "protocol=https
host=<detected-host>
username=<author>
password=<token>
"   | git credential approve

# 3 — verify immediately
git ls-remote <remote> HEAD
```

If verification succeeds:
```
┌─────────────────────────────────────────┐
│  ✅ Authenticated with github.com       │
│  Token stored in OS keychain            │
│  You won't be asked again on this       │
│  machine.                               │
└─────────────────────────────────────────┘
```

If verification fails, Claude shows the exact error and tells the user what to fix:
```
❌ Authentication failed

  401 Unauthorized → token expired or missing "repo" scope
                     generate a new one and paste it here

  403 Forbidden    → token has read-only scope
                     regenerate with "repo" checked

  Repository not   → check the URL you pasted in Phase 1
  found              make sure the repo exists and is accessible
```

Claude waits for the user to fix the issue and re-pastes the token.
Does not continue to Phase 3 until auth is confirmed working.

---

#### PHASE 3 — Build project context (interview)

Claude says:
```
┌─────────────────────────────────────────────────────────────────┐
│  DISTRIBUTED MODE — Phase 3 of 4: Build your project context   │
├─────────────────────────────────────────────────────────────────┤
│  I'll scan your project first, then ask about what I           │
│  couldn't detect. One question at a time.                       │
└─────────────────────────────────────────────────────────────────┘
```

```bash
# Scan codebase silently
ls -la
cat package.json 2>/dev/null || cat requirements.txt 2>/dev/null || cat Cargo.toml 2>/dev/null
cat README.md 2>/dev/null | head -60
cat CLAUDE.md 2>/dev/null
```

Then runs the one-question-at-a-time interview (see `references/init-interview.md`).
Also asks (as its own separate question turn):
```
What's your name or git handle?
This appears in the team changelog whenever you save context.
```

---

#### PHASE 4 — Push context to shared repo

Claude says:
```
┌─────────────────────────────────────────────────────────────────┐
│  DISTRIBUTED MODE — Phase 4 of 4: Push to shared repo         │
└─────────────────────────────────────────────────────────────────┘
```

```bash
# Write permissions file first
mkdir -p .contextit
cat > .contextit/settings.json << 'EOF'
{
  "permissions": {
    "allow": ["Write(.contextit/**)", "Edit(.contextit/**)", "Write(CLAUDE.md)"]
  }
}
EOF

# Write all context files
# stack.md, architecture.md, ui.md, features.md,
# decisions.md, conventions.md, guardrails.md, .contextit.json

# Init git and push
cd .contextit && git init
git remote add origin <remote>
git add .
git commit -m "claudeit: initial project context"
git push -u origin main

# Extend CLAUDE.md
echo -e "
## Project Context
Full SDLC context in \`.contextit/\`. Run \`claudeit load\` at session start." >> ../CLAUDE.md
```

Confirm to user:
```
┌─────────────────────────────────────────────────────────────────┐
│  ✅ claudeit initialized (DISTRIBUTED)                          │
├─────────────────────────────────────────────────────────────────┤
│  Context repo:  https://github.com/your-org/project-context    │
│  Stack:         [detected]                                      │
│  UI:            [detected]                                      │
│  Features:      [N captured]                                    │
│  Decisions:     [N logged]                                      │
│  Conventions:   [detected]                                      │
│  Guardrails:    [N set]                                         │
├─────────────────────────────────────────────────────────────────┤
│  To onboard a teammate, give them this command:                 │
│                                                                 │
│  claudeit join https://github.com/your-org/project-context.git │
│                                                                 │
│  That's all they need. claudeit handles the rest.              │
└─────────────────────────────────────────────────────────────────┘
```

---

### `claudeit join <remote-url>` (DISTRIBUTED — Team Member)

When a teammate wants to join an existing distributed project.
Claude walks them through three phases.

---

#### PHASE 1 — Authenticate

Claude says:
```
┌─────────────────────────────────────────────────────────────────┐
│  JOINING distributed project — Phase 1 of 3: Authenticate     │
├─────────────────────────────────────────────────────────────────┤
│  I need a Personal Access Token to access the context repo.    │
│                                                                 │
│  GitHub:                                                        │
│    github.com → Settings → Developer settings                  │
│    → Personal access tokens → Tokens (classic)                 │
│    → Generate new token → Scope: ✅ repo → Generate            │
│    → Copy the token (shown once only)                          │
│                                                                 │
│  GitLab:                                                        │
│    gitlab.com → Preferences → Access Tokens                    │
│    → New token → ✅ read_repository ✅ write_repository        │
│    → Create → Copy (shown once only)                           │
│                                                                 │
│  Paste your token here.                                         │
└─────────────────────────────────────────────────────────────────┘
```

Runs same credential store setup and verification as init Phase 2.
Does not continue until auth is confirmed.

---

#### PHASE 2 — Clone context repo

```bash
# Write permissions first
mkdir -p .contextit
cat > .contextit/settings.json << 'EOF'
{
  "permissions": {
    "allow": ["Write(.contextit/**)", "Edit(.contextit/**)", "Write(CLAUDE.md)"]
  }
}
EOF

# Clone shared context into .contextit/
git clone <remote-url> .contextit

# Extend CLAUDE.md
echo -e "
## Project Context
Full SDLC context in \`.contextit/\`. Run \`claudeit load\`." >> CLAUDE.md
```

Also asks (as its own question turn):
```
What's your name or git handle?
This appears in the team changelog when you save context.
```

Saves author to `.contextit/.contextit.json`.

---

#### PHASE 3 — Load and brief

Immediately runs `claudeit load` — pulls latest, reads all files, delivers briefing:

```
┌─────────────────────────────────────────────────────────────────┐
│  ✅ Joined project — you're fully context-aware                 │
├─────────────────────────────────────────────────────────────────┤
│  Stack:         [summary]                                       │
│  UI:            [theme summary]                                 │
│  Now building:  [current feature]                               │
│  Last decision: [most recent]                                   │
│  Key guardrails:[top 2]                                         │
├─────────────────────────────────────────────────────────────────┤
│  What are we working on today?                                  │
└─────────────────────────────────────────────────────────────────┘
```

No onboarding conversation needed. Teammate is ready immediately.

---

### `claudeit load` (DISTRIBUTED)

Pulls latest from remote first, then reads and briefs:

```bash
git -C .contextit pull origin main --quiet

for f in stack architecture ui features decisions conventions guardrails; do
  cat .contextit/$f.md 2>/dev/null
done
```

If pull fails (auth expired, network issue) Claude tells the user exactly:
```
⚠️  Could not pull latest context from remote.

  Token expired?  → run: claudeit login reset
  Network issue?  → check connection and try again
  Repo moved?     → update remote in .contextit/.contextit.json
```

If teammates pushed updates since last session:
```
┌─────────────────────────────────────────────────────────────────┐
│  📦 claudeit loaded — [Project Name]                           │
│  🔄 Updated since your last session:                           │
│     decisions.md  → 2 new decisions by @teammate               │
│     features.md   → auth marked as shipped                     │
├─────────────────────────────────────────────────────────────────┤
│  Stack:         [summary]                                       │
│  UI:            [theme]                                         │
│  Now building:  [current]                                       │
│  Guardrails:    [active]                                        │
├─────────────────────────────────────────────────────────────────┤
│  What are we working on today?                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

### `claudeit save` (DISTRIBUTED)

Same as SOLO — shows diff, waits for confirmation — then commits and pushes:

```bash
git -C .contextit add .
git -C .contextit commit -m "claudeit: [summary] [@author]"
git -C .contextit push origin main
```

```
┌──────────────────────────────────────────────────────┐
│  ✅ Saved and pushed                                 │
│  Your teammates will see this on their next          │
│  claudeit load.                                      │
└──────────────────────────────────────────────────────┘
```

If push fails:
```
⚠️  Push failed.

  Auth expired?   → claudeit login reset
  Conflict?       → run: git -C .contextit pull origin main
                    then try claudeit save again
```

---

### `claudeit login reset` (DISTRIBUTED)

Clears expired token and sets up a new one:

```bash
printf "protocol=https
host=<detected-host>
" | git credential reject
git config --global --unset credential.helper
```

Then re-runs the token prompt from Phase 2, stores new token, verifies.

## Shared Commands (both modes)

### `claudeit update <file> <change>`
Mid-session update to a specific context file without waiting for save.
```
claudeit update ui "switched from Inter to Geist font"
claudeit update guardrails "no third-party analytics — privacy first"
claudeit update features "payment flow — shipped"
```
Claude writes the change immediately and confirms.

### `claudeit status`
Quick snapshot:
```bash
cat .contextit/features.md
cat .contextit/.contextit.json
tail -20 .contextit/decisions.md
```
Reports: mode, features summary (shipped/building/planned), last save date, any TODO items.

---

## What Claude Knows After Load

| File | What Claude knows |
|---|---|
| `stack.md` | Languages, frameworks, libraries, infra, versions |
| `architecture.md` | System design, data flow, key patterns, folder structure |
| `ui.md` | Theme, colors, fonts, component library, conventions — writes matching UI code |
| `features.md` | What's shipped, building, planned, dropped |
| `decisions.md` | What was decided and why — never re-suggests rejected approaches |
| `conventions.md` | Naming, file structure, git, testing — follows automatically |
| `guardrails.md` | What NOT to build — flags conflicts proactively |
| `responsible-ai.md` | Six core AI principles — conflicts require written justification, logged permanently |

Guardrail enforcement during session:
```
⚠️ Guardrail conflict: "Add email notifications"
   Rule: "No email — in-app notifications only"
   Override? [yes with reason / no]
```

---

## Session Lifecycle

```
SOLO                              DISTRIBUTED
────────────────────────────      ──────────────────────────────────
claudeit load                     claudeit load
  read .contextit/ files            git pull latest from remote
  brief: stack·ui·features          read .contextit/ files
  "What are we working on?"         highlight what teammates changed
                                    brief: stack·ui·features

work normally                     work normally
claudeit update [if needed]       claudeit update [if needed]

claudeit save                     claudeit save
  show diff                         show diff
  write .contextit/ files           write .contextit/ files
  "Saved."                          git commit + push
                                    "Saved. Team sees this on next load."
```

---


---

## Responsible AI Framework (opt-in)

claudeit embeds a Responsible AI framework as an opt-in during `claudeit init`.
It lives in `.contextit/responsible-ai.md` and is enforced by Claude every session
alongside `guardrails.md`. Full guide: `references/responsible-ai-guide.md`

---

### Opt-in during `claudeit init`

After the guardrails interview, Claude asks one question (its own turn):

```
┌─────────────────────────────────────────────────────────────────┐
│  Responsible AI Framework                                       │
├─────────────────────────────────────────────────────────────────┤
│  Would you like to add a Responsible AI framework?              │
│                                                                 │
│  Six locked core principles — Transparency, Privacy,            │
│  Fairness, Human Control, Safety, Accountability —              │
│  checked against every feature and decision, every session.     │
│                                                                 │
│  Conflicts require a written justification before continuing.   │
│  Overrides are logged permanently.                              │
│                                                                 │
│  You can add project-specific principles on top.                │
│  Core principles cannot be removed.                             │
│                                                                 │
│  Add Responsible AI framework? (yes / no)                       │
└─────────────────────────────────────────────────────────────────┘
```

**If NO** → Claude logs it as an explicit decision in `decisions.md` and moves on.

**If YES** → Claude asks one follow-up (its own separate turn):
```
Any project-specific AI principles to add on top of the core six?
For example: "No AI-generated content in medical contexts without
clinical review" or "All AI suggestions reviewed by a human before
going live."

Type them now, or press enter to skip.
```

Then writes `.contextit/responsible-ai.md` and confirms:
```
┌─────────────────────────────────────────────────────────────────┐
│  ✅ Responsible AI framework added                              │
│  6 core principles locked                                       │
│  [N] project principles added                                   │
│  Active from this session onwards                               │
└─────────────────────────────────────────────────────────────────┘
```

---

### The Six Core Principles (locked for all projects)

| Principle | What Claude checks |
|---|---|
| **Transparency** | AI-generated content presented to users without disclosure |
| **Privacy** | Data collection, tracking, third-party sharing without consent |
| **Fairness** | Features that could disadvantage user groups or introduce bias |
| **Human Control** | AI actions taken without user confirmation or review |
| **Safety** | High-stakes outputs without disclaimers or human review gates |
| **Accountability** | AI-influenced decisions with no audit trail or explanation |

Core principles are **locked** — they cannot be deleted, only overridden
with a written justification that is permanently logged.

---

### Conflict Enforcement

When Claude detects a conflict during a session:

```
┌─────────────────────────────────────────────────────────────────┐
│  ⚠️  Responsible AI Conflict                                    │
├─────────────────────────────────────────────────────────────────┤
│  Proposed:   "Auto-send AI-drafted emails without user review"  │
│  Principle:  Human Control                                      │
│  Rule:       Consequential actions require human confirmation.  │
│              No autonomous AI actions without human in loop.   │
├─────────────────────────────────────────────────────────────────┤
│  Provide a written justification to proceed, or type CANCEL.   │
│  Your justification will be logged permanently.                 │
└─────────────────────────────────────────────────────────────────┘
```

Claude waits. Developer must type a real justification — not "yes", "ok",
or "approved". Claude rejects thin responses:

```
❌ "It's fine"              → not a justification
❌ "User agreed to TOS"     → not specific enough
❌ "We need it to ship"     → not a principle-based reason

✅ "Emails are drafts only — user reviews and confirms before
   send. Auto refers to drafting only. Human control over
   sending is fully preserved."
```

Once valid, Claude appends to `responsible-ai.md`:

```markdown
## YYYY-MM-DD: Override — Human Control
**Feature:** Auto-draft emails
**Conflict:** Autonomous AI action without human confirmation
**Justification:** Drafting only — user confirms before send.
**Logged by:** @author
```

---

### `claudeit update responsible-ai "<principle>"`

Add a project-specific principle at any time:
```
claudeit update responsible-ai "No AI content in patient-facing
outputs without clinical review"
```

Project principles behave like core principles — conflicts require
justification and are logged. Unlike core principles, they can be
removed by the project owner with a logged reason.

---

### Session Load with Responsible AI active

```
┌─────────────────────────────────────────────────────────────────┐
│  📦 claudeit loaded — [Project Name]                           │
│  🛡️  Responsible AI: 6 core + [N] project principles active    │
├─────────────────────────────────────────────────────────────────┤
│  Stack:  ...                                                    │
└─────────────────────────────────────────────────────────────────┘
```

Claude holds all principles in active memory and checks them against
every new feature, suggestion, or decision — without being asked.

## Context Hygiene

1. `decisions.md` and `guardrails.md` — append-only, never edited
2. Guardrail overrides need a dated reason — original never deleted
3. `features.md` — max 3 items in "building" to maintain focus
4. Files over 300 lines → Claude archives old entries automatically
5. No secrets ever written to `.contextit/`

---

## Git Integration

claudeit includes a full smart git layer — see `references/git-skill.md` for the complete reference.
Every git command runs with project context: commit messages from your session, branch
names from your conventions, PR descriptions from your diff, pre-push guardrail checks.

All write operations require confirmation. Destructive commands require typing CONFIRM.

```
claudeit commit      ← generates message from session context
claudeit push        ← guardrail check → confirm → push
claudeit pull        ← pull + plain English summary
claudeit pr          ← drafts full PR description
claudeit branch <x>  ← name enforced from conventions.md
claudeit sync        ← pull + merge + push safely in sequence
claudeit log         ← plain English git history
claudeit diff        ← diff explained in project context
claudeit git <cmd>   ← any git command with claudeit intelligence
```

---

## Quick Reference

| Command | What it does |
|---|---|
| `claudeit` | Entry point — load if initialized, setup if not |
| `claudeit init` | First-time setup (asks SOLO or DISTRIBUTED) |
| `claudeit load` | Start of session — read context, brief Claude |
| `claudeit save` | End of session — update context files |
| `claudeit update <file> <change>` | Mid-session context update |
| `claudeit update responsible-ai "<principle>"` | Add a project-specific AI principle |
| `claudeit status` | Project snapshot |
| `claudeit join <url>` | Join a distributed project |
| `claudeit login` | Set up git auth (SSH or HTTPS token) |

---

## Reference Files

- `references/templates.md` — File templates for all `.contextit/` files
- `references/init-interview.md` — How Claude extracts context during setup
- `references/guardrails.md` — How to write and enforce good guardrails
- `references/responsible-ai-guide.md` — Full Responsible AI framework, principles, and enforcement rules
- `references/git-skill.md` — Full smart git integration — all commands, smart features, and safety rules
- `references/git-token-setup.md` — Step-by-step PAT setup for GitHub/GitLab on macOS and Windows
