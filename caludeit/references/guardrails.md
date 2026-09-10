# Guardrails Guide

Guardrails are the most powerful part of contextit. They stop Claude from suggesting
things the team has already decided against — saving the same conversation from
happening in every session.

---

## What Makes a Good Guardrail

### Specific + reasoned
❌ "Keep it simple"
✅ "No feature requiring more than 2 clicks from home — complexity kills retention for our user base"

❌ "Don't add bloat"
✅ "No new npm dependency over 10kb gzipped without team discussion — bundle size is a product value"

A guardrail without a reason gets questioned. A guardrail with a reason gets respected.

---

## Four Categories

### 1. Scope — features off the table
"No social features (likes, follows, shares) — this is a thinking tool, not a network"
"No mobile app — web only until we hit 10k users"

### 2. Technical — hard constraints
"No microservices until monolith hits actual limits — premature splitting kills velocity"
"SQLite only for local storage — no external DB dependency for solo users"
"No runtime deps beyond what's in package.json at v1.0 without explicit decision"

### 3. UI/UX — patterns banned
"No modals or popups — use inline UI or slide-overs only"
"No onboarding tours — product must be self-evident"
"No infinite scroll — paginated lists only"

### 4. Ethical / product — lines not crossed
"No telemetry without explicit opt-in — off by default"
"No dark patterns — no fake urgency, no confirm-shaming, no hidden unsubscribe"
"No ads ever"

---

## How Claude Enforces Guardrails

During every session Claude checks new suggestions against `guardrails.md`.
When a conflict is detected:

```
⚠️ Guardrail conflict

Suggestion: "Add email notifications for comments"
Conflicts with: "No email — we respect user attention. In-app only."
  (set YYYY-MM-DD)

Options:
  [A] Drop it — guardrail stands
  [B] Modify — e.g. "in-app notification only, no email"
  [C] Override — requires a reason logged to guardrails.md
```

---

## Overriding a Guardrail

Guardrails are never deleted. If a guardrail needs to change, Claude appends an
override note with a date and reason. The original stays visible:

```markdown
## No user accounts — local-first tool
Local-first means no server dependency. Adding accounts changes the entire trust model.

> OVERRIDE 2025-06-01 [@author]: Adding optional cloud sync requires accounts.
> Scope: optional, opt-in, zero-knowledge only. Local-first preserved as default.
```

---

## Guardrail Decay Warning

If guardrails.md hasn't been updated in 90+ days, Claude notes on load:
"Guardrails haven't been reviewed in 90 days — still accurate?"
This prevents stale guardrails from blocking valid work.
