# Responsible AI Framework Guide

claudeit embeds a Responsible AI framework as an opt-in during `claudeit init`.
It lives in `.contextit/responsible-ai.md` and is enforced by Claude every session
alongside `guardrails.md`.

---

## How It Works

During `claudeit init`, after the guardrails interview, Claude asks one question:

```
┌─────────────────────────────────────────────────────────────────┐
│  Responsible AI Framework                                       │
├─────────────────────────────────────────────────────────────────┤
│  Would you like to add a Responsible AI framework to this       │
│  project?                                                       │
│                                                                 │
│  This gives your project six locked core principles —           │
│  Transparency, Privacy, Fairness, Human Control, Safety,        │
│  and Accountability — that Claude checks every new feature      │
│  and decision against, automatically.                           │
│                                                                 │
│  If something conflicts, Claude requires a written              │
│  justification before you can continue. The override is         │
│  logged permanently to responsible-ai.md.                       │
│                                                                 │
│  You can add your own principles on top of the core six.        │
│  The core six cannot be removed — only overridden with          │
│  a logged reason.                                               │
│                                                                 │
│  Add Responsible AI framework? (yes / no)                       │
└─────────────────────────────────────────────────────────────────┘
```

If NO → Claude logs this as an explicit decision:
```markdown
## YYYY-MM-DD: Responsible AI framework declined at init
Explicit decision. Can be added later by creating .contextit/responsible-ai.md
and running `claudeit update`.
```

If YES → Claude asks one follow-up question (its own turn):
```
Do you have any project-specific AI principles to add on top of
the core six? For example: "All AI suggestions must be reviewed
by a human before going live" or "No AI-generated content in
medical or legal contexts."

Type them now, or press enter to skip.
```

Then writes `.contextit/responsible-ai.md` and confirms.

---

## The Six Core Principles

These are locked. They appear in every project that opts in.
They cannot be deleted — only overridden with a dated, logged reason.

### 1. Transparency
Users must always know when AI has generated, influenced, or ranked content.
No hidden automation. No AI outputs presented as human-authored without disclosure.

**Claude checks:**
- New features that surface AI-generated content to users
- Any output that could be mistaken as human-authored
- Recommendation systems or ranking that uses AI without disclosure

### 2. Privacy
Data minimisation and user consent are non-negotiable.
Collect only what is strictly necessary. Never share or sell user data.
No behavioural tracking without explicit, informed, opt-in consent.

**Claude checks:**
- Features that collect, store, or transmit user data
- Analytics or tracking integrations
- Personalisation features that profile user behaviour
- Any third-party data sharing

### 3. Fairness
Features must work equitably across user groups.
No algorithmic bias. No design that disadvantages users based on
race, gender, age, disability, location, or socioeconomic status.

**Claude checks:**
- Personalisation or recommendation features
- Features that rank or score users
- Content moderation or filtering logic
- Any ML model integration

### 4. Human Control
AI assists — humans decide. Final consequential actions always
require human confirmation. Users can always override, undo,
or opt out of AI-driven behaviour. No fully autonomous AI actions
that affect users without a human in the loop.

**Claude checks:**
- Any AI action taken without user confirmation
- Automated decisions with real-world consequences
- Features that remove or reduce human review steps
- AI that acts on behalf of a user without explicit instruction

### 5. Safety
No feature that could cause harm if the AI output is wrong.
High-stakes outputs — medical, legal, financial, safety-critical —
must carry explicit disclaimers and human review gates.
Failure modes must be considered before building, not after.

**Claude checks:**
- Features in high-stakes domains
- AI outputs that users might act on without verification
- Error handling — what happens when the AI is wrong?
- Any AI feature with irreversible consequences

### 6. Accountability
Decisions made with or by AI must be logged and explainable.
If an AI-influenced decision causes harm, there must be a clear
record of what happened and why. No black-box decisions that
cannot be audited or explained to an affected user.

**Claude checks:**
- Features that make or influence decisions affecting users
- Logging and audit trail coverage of AI actions
- Whether affected users can request an explanation
- Whether the team can reconstruct what happened post-incident

---

## Conflict Enforcement

When Claude detects a conflict with any principle during a session:

```
┌─────────────────────────────────────────────────────────────────┐
│  ⚠️  Responsible AI Conflict                                    │
├─────────────────────────────────────────────────────────────────┤
│  Proposed:   "Auto-send AI-drafted emails without user review"  │
│  Principle:  Human Control                                      │
│  Rule:       Final consequential actions require human          │
│              confirmation. No autonomous AI actions that        │
│              affect users without a human in the loop.         │
├─────────────────────────────────────────────────────────────────┤
│  To proceed you must provide a written justification.           │
│  This will be logged permanently to responsible-ai.md.         │
│                                                                 │
│  Type your justification, or type CANCEL to drop the feature.  │
└─────────────────────────────────────────────────────────────────┘
```

Claude waits. The developer must type a justification — not just "yes" or "ok".
A justification must explain WHY the conflict is acceptable in this context.

Claude rejects thin justifications:
```
❌  "It's fine"           → not a justification
❌  "User agreed to TOS"  → not specific enough
❌  "We need it to ship"  → not a principle-based reason

✅  "Emails are drafts only — user sees and edits before send,
    confirmation is built into the send flow. The 'auto' refers
    to drafting, not sending. Human review is preserved."
```

Once a valid justification is provided, Claude logs it:

```markdown
## YYYY-MM-DD: Override — Human Control
**Feature:** Auto-draft emails without user review
**Conflict:** Autonomous AI action without human confirmation
**Justification:** Emails are drafts only. User reviews and
confirms before send. Auto refers to drafting only — human
control over sending is preserved.
**Logged by:** @author
```

---

## Project-Specific Principles

Added during init or at any time with:
```
claudeit update responsible-ai "principle: explanation"
```

Example:
```
claudeit update responsible-ai "No AI-generated content in
patient-facing outputs without clinical review"
```

Project-specific principles behave exactly like core principles —
conflicts require a written justification and are logged.
But unlike core principles, they CAN be removed by the project
owner with a logged reason.

---

## responsible-ai.md File Structure

```markdown
# Responsible AI Framework — [Project Name]
> Opted in: YYYY-MM-DD | Author: @handle

## Core Principles (locked)
[Six principles — written by claudeit, not edited by hand]

## Project Principles (customisable)
[Added by the team — can be removed with logged reason]

## Override Log
[Append-only — every conflict justification logged here]
```

---

## Loading at Session Start

When `claudeit load` runs and `responsible-ai.md` exists, Claude
includes it in the session briefing:

```
┌─────────────────────────────────────────────────────────────────┐
│  📦 claudeit loaded — [Project Name]                           │
│  🛡️  Responsible AI framework active (6 core + 2 project)      │
├─────────────────────────────────────────────────────────────────┤
│  Stack:  ...                                                    │
│  ...                                                            │
└─────────────────────────────────────────────────────────────────┘
```

Claude holds all six principles in active session memory and
checks them against every new feature, suggestion, or decision
without being asked.

---

## What Never Happens

- Core principles are never silently deleted
- A conflict is never bypassed without a written justification
- Justifications are never edited after being logged
- "yes", "ok", or "approved" are never accepted as justifications
- The framework is never disabled mid-session
