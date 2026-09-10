# Init Interview Guide

One question at a time. Always. Wait for the answer before asking the next one.
Never group questions. Never use bullet lists of questions. Never say "I need a few details".

---

## The Rule

Ask ONE question. Wait. Get the answer. Ask the NEXT question.

This applies even if multiple unknowns exist. Patience over speed.
A focused interview feels like a conversation, not a form.

---

## Phase 1: Auto-Detection (Claude does this silently, no output)

Before asking anything, scan the codebase:

```bash
ls -la
cat package.json 2>/dev/null
cat requirements.txt 2>/dev/null
cat Cargo.toml 2>/dev/null
cat go.mod 2>/dev/null
cat README.md 2>/dev/null | head -60
cat CLAUDE.md 2>/dev/null
```

From the scan, Claude builds a list of what it KNOWS and what it NEEDS TO ASK.
Only unknown items become questions. Already-detected items are never asked about.

---

## Phase 2: One Question at a Time

Questions are asked in this order. Skip any that were already detected.

---

### Q1 — Project name and purpose

Only ask if not found in README or package.json.

```
What is this project? One sentence — what it does and who uses it.
```

Wait for answer. Move to Q2.

---

### Q2 — Tech stack gaps

Only ask about pieces not detected in the scan.
Frame it around what was found:

```
I can see you're using [detected things]. What about [first missing piece]?
```

Ask about ONE missing piece at a time.
If multiple pieces are missing, ask about them one by one across separate turns.

Examples:
- "What database are you using?"
- "Where is this hosted or deployed?"
- "What's your package manager — npm, pnpm, or yarn?"

Wait for each answer before asking the next stack question.

---

### Q3 — UI and theme

Only ask if no UI library or styling approach was detected.

```
What's your UI setup — component library, styling approach, and rough color vibe?
Even loose is fine: "dark, minimal, Tailwind" is enough.
```

Wait for answer. Move to Q4.

---

### Q4 — Features

```
What features are already shipped and working?
```

Wait for answer. Then:

```
What are you actively building right now?
```

Wait for answer. Then:

```
What's planned next after that?
```

Wait for answer. Move to Q5.

---

### Q5 — Major decisions

```
Any major technical or product decisions already locked in?
Things like "we chose X over Y" or "we decided not to build Z".
```

Wait for answer.
If they mention something, follow up once: "Why that choice?" — then move on.
Move to Q6.

---

### Q6 — Coding conventions

```
Any coding conventions I should follow in this project?
Things like file naming, folder structure, commit format, or code style.
```

Wait for answer. Move to Q7.

---

### Q7 — Guardrail 1 (scope)

```
What features have you already decided NOT to build — even if someone asks for them?
```

Wait for answer. Move to Q8.

---

### Q8 — Guardrail 2 (technical)

```
Any hard technical constraints every piece of code must respect?
For example: bundle size limits, banned dependencies, backwards compatibility rules.
```

Wait for answer. Move to Q9.

---

### Q9 — Guardrail 3 (UI/product)

```
Any UI patterns or product behaviors that are completely off the table?
For example: no modals, no dark patterns, no third-party tracking.
```

Wait for answer.

---

## Phase 3: Write Files (silent)

After all questions are answered, Claude writes all `.contextit/` files without
narrating each one. Then shows a single confirmation summary:

```
┌─────────────────────────────────────────────────┐
│  ✅ claudeit initialized                         │
├─────────────────────────────────────────────────┤
│  Stack        Next.js 14 · TypeScript · Prisma  │
│  UI           Shadcn/ui · dark · Inter font      │
│  Features     2 shipped · 1 building · 3 planned │
│  Decisions    1 logged                           │
│  Conventions  kebab-case · conventional commits  │
│  Guardrails   2 set                              │
├─────────────────────────────────────────────────┤
│  Anything missing or wrong? Tell me and          │
│  I will update it before we start.               │
└─────────────────────────────────────────────────┘
```

Wait for corrections. Apply any changes. Then session begins.

---

## Handling Empty Projects

If the project directory is empty (no files detected), do NOT say
"The project directory is currently empty. Before I write context files,
I need a few details:" and then list questions.

Instead, say:

```
┌────────────────────────────────────────────────────┐
│  New project — let's build your context together.  │
│  I'll ask you a few things, one at a time.         │
└────────────────────────────────────────────────────┘
```

Then start with Q1 and go one question at a time from there.

---

## Handling Partial Answers

If the developer says "not sure yet" or "TBD":
- Accept it without pushing
- Write `<!-- TBD -->` in that section
- Move to the next question
- Partial context is better than no context

---

## What Never Happens

- ❌ Never list multiple questions at once
- ❌ Never say "I need a few details" followed by a list
- ❌ Never use numbered or bulleted question lists
- ❌ Never ask about something already detected in the scan
- ❌ Never ask all guardrail questions in one turn
