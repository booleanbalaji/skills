# Project Brain — File Templates

Use these templates verbatim when creating a new brain. Replace `<Project Name>` with the user's project name. Leave `(fill in)` markers wherever the user hasn't supplied a fact — do not invent.

---

## OV — Overview

Filename: `01-OV-overview.md`

```markdown
# [OV] Overview

**What this project is:** (one sentence — what it does, not how)

**Why it exists:** (the problem or motivation)

**Who it's for:** (intended user / audience)

**Current status:** (idea / prototype / shipping / archived)
```

---

## GO — Goals & Non-Goals

Filename: `02-GO-goals.md`

```markdown
# [GO] Goals & Non-Goals

## Goals
What success looks like. Concrete where possible.

- (fill in)

## Non-Goals
Things we are deliberately NOT doing. This saves AI from suggesting them.

- (fill in)
```

---

## AR — Architecture

Filename: `03-AR-architecture.md`

```markdown
# [AR] Architecture

**Stack:** (languages, frameworks, services)

**Key components:**
- (fill in)

**Data model / important entities:**
- (fill in)

**External dependencies:**
- (fill in)

**How things connect:** (optional — short prose or ASCII sketch if it helps)
```

---

## DC — Decisions

Filename: `04-DC-decisions.md`

```markdown
# [DC] Decisions

A log of choices made and why. Newest at the top. Never edit old entries.

<!-- Template for a new entry:

## YYYY-MM-DD — <Decision title>
**Context:** what was happening
**Choice:** what we picked
**Why:** the reasoning
**Alternatives considered:** what we didn't pick

-->
```

---

## ST — Current State

Filename: `05-ST-state.md`

```markdown
# [ST] Current State

Snapshot of right now. Rewrite freely as things change.

**What works:**
- (fill in)

**In progress:**
- (fill in)

**Broken / known issues:**
- (fill in)

**Next 3 things to do:**
1. (fill in)
2. (fill in)
3. (fill in)
```

---

## GL — Glossary

Filename: `06-GL-glossary.md`

```markdown
# [GL] Glossary

Project-specific terms an AI won't know from general knowledge. One sentence per term.

- **(term)** — (definition)
```

---

## OQ — Open Questions

Filename: `07-OQ-questions.md`

```markdown
# [OQ] Open Questions

Things I'm unsure about. Useful when asking AI to help think through them. When a question is resolved, move it to DC as a decision.

- (fill in)
```

---

## README

Filename: `README.md`

```markdown
# <Project Name> — Project Brain

This folder is the canonical memory of the **<Project Name>** project. It exists so any AI session (Claude, Claude Code, Codex, ChatGPT) can be brought up to speed instantly by pasting in the bundled context.

## Files

- `01-OV-overview.md` — what this project is
- `02-GO-goals.md` — goals and non-goals
- `03-AR-architecture.md` — stack and structure
- `04-DC-decisions.md` — log of choices and why (append, don't edit)
- `05-ST-state.md` — current snapshot (rewrite freely)
- `06-GL-glossary.md` — project-specific terms (optional)
- `07-OQ-questions.md` — open questions (optional)
- `.history/` — timestamped snapshots of past ST files (auto-created on state updates; used for diffs)

## Workflow

**Before an AI session:** concatenate the files you want into context and paste them in. Either run the compile step (Claude can do this with the project-brain skill), or paste the relevant files directly.

**After an AI session:** update ST with what changed. Log any meaningful choices in DC. Add new project-specific terms to GL.

## Rules

- One job per file. Don't blur Decisions, State, and Architecture.
- Append to DC — never rewrite old entries. It's a log.
- ST is the one file you rewrite freely. It's a snapshot, not history.
- Short over complete. A small, current brain beats a large, stale one.
```
