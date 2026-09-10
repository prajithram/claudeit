# .contextit/ File Templates

Claude uses these templates when running `contextit init` or `contextit init dctx`.
It pre-fills what it can detect from the codebase and asks targeted questions for the rest.

---

## .contextit.json

```json
{
  "project": "My Project",
  "mode": "sctx",
  "author": "dev-handle",
  "remote": "",
  "created": "YYYY-MM-DD",
  "last_save": "YYYY-MM-DD"
}
```

---

## stack.md

```markdown
# Stack — [Project Name]
> Last updated: YYYY-MM-DD

## Language & Runtime
- Language: TypeScript / Python / Rust / ...
- Runtime: Node 20 / Python 3.12 / ...

## Framework
- [Framework name] — why this was chosen

## Key Libraries
| Library | Purpose |
|---|---|
| [lib] | [what it does in this project] |

## Database
- [DB name + why]

## Infrastructure
- Hosting: [where it runs]
- CI/CD: [pipeline tool]
- Other: [CDN, queues, caches, etc.]

## Package Manager
- [npm / pnpm / yarn / pip / cargo / etc.]

## Versions to Know
- Node: X.X
- [Framework]: X.X
```

---

## architecture.md

```markdown
# Architecture — [Project Name]
> Last updated: YYYY-MM-DD

## Overview
[2-3 sentences: what the system does and how it fits together]

## System Diagram
[ASCII or description of key components and how they connect]

## Data Flow
[How a typical request moves through the system]

## Key Patterns
- [Pattern name]: [how and where it's used]

## Folder Structure
```
src/
├── [folder]  ← [what lives here]
```

## External Integrations
| Service | Purpose | Auth method |
|---|---|---|

## Known Constraints
- [Any hard technical limits developers must work within]
```

---

## ui.md

```markdown
# UI Context — [Project Name]
> Last updated: YYYY-MM-DD

## Component Library
- [Shadcn/ui / MUI / Radix / custom / etc.]
- Installation: [how components are added to this project]

## Styling
- Approach: [Tailwind / CSS Modules / Styled Components / etc.]
- Dark mode: [supported / not supported / default]

## Theme

### Colors
| Token | Value | Usage |
|---|---|---|
| Primary | #XXXXXX | [buttons, links, CTAs] |
| Background | #XXXXXX | [main bg] |
| Surface | #XXXXXX | [cards, panels] |
| Text | #XXXXXX | [body text] |
| Muted | #XXXXXX | [secondary text, placeholders] |
| Accent | #XXXXXX | [highlights, badges] |
| Destructive | #XXXXXX | [errors, delete actions] |

### Typography
| Role | Font | Size | Weight |
|---|---|---|---|
| Body | [font] | [size] | [weight] |
| Heading | [font] | [size] | [weight] |
| Code | [font] | [size] | [weight] |

## Layout
- Max content width: [px or container class]
- Base spacing unit: [px or Tailwind scale]
- Breakpoints: [mobile-first / desktop-first, key breakpoints]

## Conventions
- Icons: [Lucide / Heroicons / custom — how to import]
- Images: [next/image / standard img / CDN pattern]
- Loading states: [skeleton / spinner / what component]
- Error states: [how errors are shown to users]
- Empty states: [how empty lists/pages look]

## Do / Don't
- ✅ DO: [key UI rule]
- ❌ DON'T: [specific pattern to avoid]
```

---

## features.md

```markdown
# Features — [Project Name]
> Last updated: YYYY-MM-DD

## ✅ Shipped
- [Feature name] — [one line description]

## 🔨 Building (max 3)
- [Feature name] — [what's being built, by whom if dctx]

## 📋 Planned
- [Feature name] — [brief description]

## ❌ Dropped
- [Feature name] — [why it was dropped]
```

---

## decisions.md

```markdown
# Decisions — [Project Name]
> Append-only. Format: `## YYYY-MM-DD: Title`

---

## YYYY-MM-DD: [First Decision Title]
**Context:** [What situation forced this decision]
**Decision:** [What was chosen]
**Why:** [The reasoning]
**Rejected alternatives:** [What else was considered and why it lost]
```

---

## conventions.md

```markdown
# Conventions — [Project Name]
> Last updated: YYYY-MM-DD

## Code Style
- Formatter: [Prettier / Black / rustfmt / etc. — config location]
- Linter: [ESLint / Ruff / Clippy / etc.]
- [Key style rules Claude should follow when writing code]

## Naming
- Files: [kebab-case / camelCase / snake_case]
- Components: [PascalCase]
- Functions: [camelCase / snake_case]
- Constants: [UPPER_SNAKE / camelCase]
- Database: [snake_case tables, etc.]

## File Structure Rules
- [Where new components go]
- [Where new API routes go]
- [Where types/interfaces go]
- [Where tests go]

## Git
- Branch naming: [feature/xxx / fix/xxx / etc.]
- Commit format: [conventional commits / custom]
- PR size: [target lines per PR]

## Testing
- Framework: [Jest / Vitest / Pytest / etc.]
- Coverage expectation: [X%]
- What must be tested: [unit / integration / e2e]
```

---

## guardrails.md

```markdown
# Guardrails — [Project Name]
> Append-only. Overrides require a dated note — originals are never deleted.

## Scope — What We're NOT Building
- [Feature explicitly out of scope + reason]

## Technical — Hard Constraints
- [Technical rule every developer must follow + reason]

## UI / UX — Patterns We Reject
- [Specific UI pattern banned + reason]

## Ethical / Product — Lines We Don't Cross
- [Business or ethical limit + reason]
```
