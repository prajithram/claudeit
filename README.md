# Claudeit

*Claude forgets. contextit remembers.*

---

Every session starts the same way. You open Claude Code, say "let's keep working on the auth flow," and Claude says — "Could you remind me of your stack?"

You explain Next.js. Again. You explain Tailwind. Again. You explain why you chose Zustand over Redux three weeks ago. Again.

contextit fixes that. One command at the start of a session, Claude knows everything. Your stack, your UI theme, your decisions, your guardrails. You never re-explain. You just work.

## Before / after

You start a session to build a new feature.

**Without contextit:**
```
You:   "let's add a payment form"
Claude: "Sure! What stack are you using?"
You:   "Next.js, Tailwind, Shadcn, Stripe..."
Claude: "Got it. What's your color theme?"
You:   "Neutral slate, dark mode default, Inter font..."
Claude: "Any conventions I should follow?"
You:   "We already decided not to use modals, everything is—"
```
Five minutes of re-explaining before a line of code.

**With contextit:**
```
claudeit load

📦 contextit loaded — Payments Platform
   Stack:        Next.js 14 · TypeScript · Prisma · Tailwind · Stripe
   UI:           Shadcn/ui · slate · dark default · Inter
   Now building: checkout redesign
   Guardrails:   no modals · no third-party analytics

What are we working on today?
```

## How it works

contextit extends Claude's built-in `CLAUDE.md` system with a `.contextit/` folder
of structured SDLC context files. Claude reads them at the start of every session.

```
your-project/
├── CLAUDE.md              ← Claude's built-in memory (unchanged)
└── .contextit/
    ├── stack.md           ← Languages, frameworks, libraries, infra
    ├── architecture.md    ← System design, data flow, patterns
    ├── ui.md              ← Theme, colors, fonts, component library
    ├── features.md        ← Shipped / building / planned / dropped
    ├── decisions.md       ← What was decided and why
    ├── conventions.md     ← Naming, structure, git, testing rules
    └── guardrails.md      ← What NOT to build — Claude enforces this
```

After `claudeit load`, Claude knows all of it. It writes UI code in your theme.
It follows your conventions. It never re-suggests things you already rejected.
It flags new ideas that conflict with your guardrails — before you build them.

## Two modes

### sctx — Solo

Just you. No git required. Context lives in `.contextit/` in your project root.
Survives session resets. Claude reads it on load, updates it on save.

```
claudeit init    ← one-time setup
claudeit load    ← start of every session
claudeit save    ← end of every session
```

### dctx — Distributed

A team. Context lives in a shared git repo. Every developer runs `claudeit load`
to pull the latest. One developer's decision becomes everyone's context on their
next load. New teammates run one command and arrive fully context-aware.

```
claudeit init           ← project owner, one time
claudeit join <url>     ← every teammate, one time
claudeit load           ← everyone, every session
claudeit save           ← everyone, every session
```

## Install

Add the skill to Claude Code:

```
/skills add contextit.skill
```

Then type `claudeit` in any session to get started.

## Usage

### First time

```
claudeit
```

Claude asks one question: **SOLO or DISTRIBUTED?**

Explains both options, waits for your answer, then branches into the right setup flow.
From there, one question at a time — never a form, always a conversation.

### Every session (solo)

```
claudeit load    ← reads .contextit/, briefs Claude
...work...
claudeit save    ← Claude shows what changed, writes files
```

### Every session (distributed)

```
claudeit load    ← git pull + reads files + briefs Claude
...work...
claudeit save    ← Claude shows diff, writes files, git push
```

Teammates see your updates on their next `claudeit load`.

### Mid-session

```
claudeit update ui "switched to Geist font"
claudeit update guardrails "no third-party analytics"
claudeit update features "auth — shipped"
claudeit status
```

## Guardrails

The most important file in `.contextit/` is `guardrails.md`.

It captures what NOT to build — scope limits, banned patterns, technical constraints,
ethical lines. Claude checks every new suggestion against it automatically.

When something conflicts:

```
⚠️  Guardrail conflict: "Add email notifications"
    Rule: "No email — in-app notifications only. We respect user attention."

    Options:
      [A] Drop it — guardrail stands
      [B] Modify — in-app only, no email
      [C] Override — requires a dated reason logged to guardrails.md
```

Guardrails are permanent. They can be overridden with a reason, never silently deleted.

## Distributed setup — what each person does

**Project owner** (once):
```
claudeit init
```
Claude walks through 4 phases: create repo on GitHub → set up token → build context → push.
At the end, Claude outputs one line to share with the team:
```
claudeit join https://github.com/your-org/project-context.git
```

**Every teammate** (once):
```
claudeit join https://github.com/your-org/project-context.git
```
Claude walks through token setup, clones the context repo, loads everything, briefs immediately.
One command. Fully context-aware from the first session.

## Authentication (distributed only)

contextit uses HTTPS + Personal Access Token. No SSH setup required.

When you run `claudeit init` or `claudeit join`, Claude detects whether you're already
authenticated. If not, it walks you through creating a token on GitHub and stores it
securely in your OS keychain — never in any file, never committed to git.

**macOS** → stored in Keychain  
**Windows** → stored in Windows Credential Manager via Git Credential Manager  
**Linux** → stored in libsecret / GNOME keyring (plaintext fallback with warning)

Token expired? Run `claudeit login reset` — Claude clears the old one and
walks you through entering a new one.

Full setup guide: [`references/git-token-setup.md`](references/git-token-setup.md)

## Commands

| Command | What it does |
|---|---|
| `claudeit` | Smart entry — loads if initialized, starts setup if not |
| `claudeit init` | First-time setup — asks SOLO or DISTRIBUTED, then guides through |
| `claudeit load` | Start of session — pull (dctx) + read context + brief Claude |
| `claudeit save` | End of session — show diff, write files, push (dctx) |
| `claudeit update <file> <change>` | Mid-session context update |
| `claudeit status` | Features snapshot, last save, open TODOs |
| `claudeit join <url>` | Join a distributed project |
| `claudeit login` | Set up or refresh git authentication |
| `claudeit login reset` | Clear expired token and re-authenticate |

## What Claude knows after load

| File | What Claude knows |
|---|---|
| `stack.md` | Languages, frameworks, libraries, infra, versions — no more "what are you using?" |
| `architecture.md` | System design, folder structure, data flow, patterns |
| `ui.md` | Theme, colors, fonts, component library — Claude writes matching UI code automatically |
| `features.md` | What's shipped, building, planned, dropped — no more "where are we?" |
| `decisions.md` | What was decided and why — Claude never re-suggests what you already rejected |
| `conventions.md` | Naming, file structure, git, testing — Claude follows your standards without being told |
| `guardrails.md` | What NOT to build — Claude flags conflicts before you start building |

## Context hygiene

- `decisions.md` and `guardrails.md` are append-only — never edited, only added to
- `features.md` max 3 items in "building" — Claude enforces focus
- Files over 300 lines → Claude archives old entries automatically
- No secrets ever written to `.contextit/` — Claude refuses and warns
- Guardrail overrides require a dated reason — original rule is never deleted

## FAQ

**Does it need a config file?**  
Yes — `.contextit/.contextit.json`. Claude creates it during init. You never edit it by hand.

**What if my project directory is empty?**  
Claude still runs the interview, one question at a time, and builds the context from
your answers. Partial context beats no context.

**What if a teammate makes a decision I disagree with?**  
Context updates go through `claudeit save` which shows a diff before committing.
In distributed mode, treat context PRs like code PRs — review before merging.

**Does it work with existing CLAUDE.md files?**  
Yes. contextit appends one pointer line to your existing `CLAUDE.md` and leaves
everything else untouched. Both systems work together.

**What if I want to add GitLab later?**  
GitLab support is on the roadmap. The token flow is identical — only the host
and scope names differ.

**Can I use it on multiple projects?**  
Yes. Each project has its own `.contextit/` folder. Completely independent.

## License

MIT.
