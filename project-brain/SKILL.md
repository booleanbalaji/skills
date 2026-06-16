---
name: project-brain
description: Create, maintain, and use a "Project Brain" — a small, structured set of markdown files that act as the single source of truth for one project, designed to be pasted into any AI tool (Claude, Claude Code, Codex, ChatGPT) as grounding context. Use whenever the user wants to set up project context for AI, build a "project brain" or "project memory" or "context pack," create a knowledge base for one specific project (not general notes), generate a context bundle to paste into AI tools, update an existing brain (log a decision, update current state, add a glossary term), or asks Claude to "remember" a project across sessions. Also trigger when the user pastes a brain bundle (sections tagged [OV], [AR], [DC], etc.) and asks for help with the project — treat those sections as grounding. Do NOT trigger for general Zettelkasten / second-brain systems or single-file READMEs.
---

# Project Brain

A Project Brain is a small set of canonical markdown files that hold the memory of **one** project. Not random notes. Not full chat history. The single source of truth you can paste into any AI tool to give it instant, grounded context.

The whole skill rests on one idea: **AI sessions start blank, so projects need persistent memory that lives outside any single tool.**

## When to use this skill

- User says "set up a project brain" / "make a brain for X" / "create AI context for my project"
- User wants to capture project context they keep re-explaining to AI tools
- User asks Claude to help maintain context across sessions for a specific project
- User pastes a brain bundle (sections tagged `[OV]`, `[AR]`, `[DC]`, etc.) and asks for help — use those sections as grounding
- User describes a workflow problem like "I keep re-explaining my project every chat" — propose a brain
- User wants to log a decision, update what's broken, or add a glossary term for a project that already has a brain

## When NOT to use this skill

- General note-taking, journaling, or Zettelkasten covering many topics → that's a different system
- A single-file README is enough → don't over-engineer
- The project is touched daily with one AI → working memory is sufficient
- The user wants a personal knowledge base across all of life → out of scope

## The Canonical Six (+ two optional)

A Project Brain has a deliberately small, fixed structure. Each file has one job. Don't blur them.

| Code | Name              | One-line purpose                                      | Default include |
|------|-------------------|-------------------------------------------------------|-----------------|
| OV   | Overview          | What this project is, why it exists, who it's for     | yes             |
| GO   | Goals & Non-Goals | What we're doing and explicitly NOT doing             | yes             |
| AR   | Architecture      | Stack, components, data model, dependencies           | yes             |
| DC   | Decisions         | Log of choices made and why (newest first)            | yes             |
| ST   | Current State     | What works, what's in progress, what's broken, next 3 | yes             |
| GL   | Glossary          | Project-specific terms AI won't know                  | optional        |
| OQ   | Open Questions    | Things I'm unsure about                               | optional        |

The two-letter codes matter. They become section tags in the compiled bundle so the AI knows where to look for what.

## Creating a brain (from scratch)

When the user asks for a new brain, follow this flow:

1. **Ask for the project name and a one-sentence description.** Don't ask more than two questions up front — fill the rest with reasonable placeholders and let them edit.
2. **Create a folder.** Default name: `<project-slug>-brain/` (e.g. `my-cli-tool-brain/`). If the user has a preferred location (existing repo, Obsidian vault), put it there.
3. **Write six files using the templates in `references/templates.md`.** Use the user's one-sentence description to pre-fill what you can in Overview; leave clear `(fill in)` placeholders elsewhere — never invent details about the project.
4. **Add a `README.md`** in the same folder that briefly explains what this folder is and how to use it (for future-self and collaborators). Template in `references/templates.md`.
5. **Present the files** with `present_files` so the user can download or open them.
6. **Tell the user the loop**: edit these files as the project evolves; before any AI session, concatenate them (or use the compile workflow below) and paste in as context.

**Do NOT** invent project details. If the user hasn't told you the stack, write `(fill in)` — making it up creates fake "memory" that misleads future AI sessions.

## Compiling the context bundle

When the user says "compile my brain" / "give me the context to paste" / "make the bundle":

1. Read all the files in the brain folder (or whichever ones the user specifies).
2. Concatenate them using this exact format:

```
# <Project Name> — Project Brain

This is the canonical context for the project "<Project Name>". It is a small set of source-of-truth files. Use it to ground your responses; don't restate it back to me unless asked.

---

## [OV] Overview

<contents of overview file, trimmed>

---

## [AR] Architecture

<contents>

---

(...and so on for each included file)
```

3. Save the bundle as `<project-slug>-brain-bundle.md` in the brain folder and present it. The user copies this whole thing into their next AI session.
4. Report rough token count (chars ÷ 4) so the user knows the context size.

The user can choose to exclude files from a bundle (e.g. skip Glossary if not relevant for the current task). Ask if they want to exclude anything; default to including OV, GO, AR, DC, ST.

## Multi-brain awareness

A user can have several brains — one folder per project. Don't assume there's only one.

**Discovering brains**: A brain folder is identifiable by containing files matching the pattern `0?-(OV|GO|AR|DC|ST|GL|OQ)-*.md` plus a `README.md` that mentions "Project Brain." When asked to do something brain-related, look for brains in this order:
1. The current working directory, if it's a brain folder
2. A `project-brains/` or `brains/` folder in the user's home or workspace
3. Wherever the user explicitly points

**Picking the right brain**: When the user's request is ambiguous about which project (e.g. "log a decision" with no project name):
- If exactly one brain exists in scope → use it, but confirm in your reply ("Logging to **my-cli-tool** brain.")
- If multiple brains exist → ask which one, listing them by name. Never guess.
- If the user names a project that doesn't have a brain yet → ask if they want to create one.

**Listing brains**: When the user says "list my brains" / "what brains do I have," scan the likely locations and show a short table: name, last-modified date of ST (their freshness signal), and number of DC entries (their activity signal). Don't dump file contents.

**Brain naming**: Folders should be `<slug>-brain/` (e.g. `linguafarm-brain/`). The slug is what the user refers to in conversation. Inside each brain, the `README.md` should start with `# <Project Name> — Project Brain` so the human-readable name is recoverable.

**Cross-brain contamination — avoid it**: When working in one brain, don't pull facts from another brain unless the user explicitly asks. Decisions, glossary terms, and architecture are project-scoped on purpose.

## Maintaining a brain

A brain is only useful if it's current. Help the user with the most common updates:

**Logging a decision (DC)**: Prepend a new dated entry to the top of Decisions. Format:
```
## YYYY-MM-DD — <Decision title>
**Context:** what was happening
**Choice:** what we picked
**Why:** the reasoning
**Alternatives considered:** what we didn't pick
```
Never reorder or rewrite old decisions — they're a log.

**Updating Current State (ST)**: This file is the *only* one that gets rewritten freely. It's a snapshot of "right now." When user says "update the state," do this in order:
1. **Archive the existing ST first.** Copy the current `05-ST-state.md` to `.history/ST-<YYYY-MM-DD-HHMM>.md` inside the brain folder, creating `.history/` if needed. This is what makes diffs possible later.
2. Ask what changed since the last edit (works / in progress / broken / next 3).
3. Rewrite the four sections in the live file.
4. Confirm in your reply: "Archived previous ST to `.history/ST-2026-06-16-1430.md` and updated state."

Don't archive on every tiny edit — only on a meaningful update. If the user is just fixing a typo, skip the archive.

## Diffing state

When the user asks "what's changed?" / "diff my state" / "what did I update since last time":

1. Read the current `05-ST-state.md`.
2. Find the most recent file in `.history/` (sort by filename — they're timestamped).
3. Compare the four sections (works / in progress / broken / next 3) and report changes as a short bulleted summary:

```
Since <date of previous snapshot>:

**Newly working:**
- <item that appeared in "works" and wasn't there before>

**Newly broken:**
- <item that appeared in "broken">

**Resolved (no longer broken):**
- <item that was in "broken" before but isn't now>

**Next 3 changed:**
- Was: <previous top item>
- Now: <new top item>
```

Keep it terse. If nothing's changed, say so. If there's no `.history/` folder yet, say "no previous snapshot — this is the first state I can see" and offer to start tracking from now.

For comparing against an older snapshot (not the most recent), let the user pick: "I have snapshots from <list of dates> — which do you want to compare against?"

**Adding a glossary term (GL)**: Append to the list. Keep definitions to one sentence.

**Refactoring Architecture (AR)**: When the stack or structure changes meaningfully, update AR — but also log the change in DC so there's a record of *why* it changed.

**Closing an Open Question (OQ)**: Move it from OQ into DC as a decision with the answer.

## Treating a pasted brain as grounding context

If the user pastes a brain bundle into the conversation (recognizable by the `[OV]`, `[AR]`, etc. section tags or the `# <Name> — Project Brain` header):

- Treat its contents as **authoritative** for facts about the project.
- Don't restate the brain back at the user unless asked — they wrote it.
- When answering, prefer brain content over general assumptions ("Per the brain's Architecture section, you're on Supabase, so...").
- If the user asks something the brain doesn't cover, say so and offer to add it (e.g. "The brain doesn't specify your auth flow — want to add it to AR?").
- If the brain contradicts itself or is out of date based on what the user just said, flag it: "Heads up — your ST says X works but you just said it's broken. Want to update ST?"

## Design principles for the brain itself

These are the rules that make a brain useful rather than just "more notes." Keep them in mind when helping the user write or edit:

1. **One job per file.** Decisions ≠ Current State ≠ Architecture. If something doesn't fit, it probably belongs in its own section or doesn't belong at all.
2. **Short over complete.** A brain that fits in 2000 tokens beats one that fits in 20,000. AI context windows aren't free, and verbose brains stop getting maintained.
3. **Non-goals are as valuable as goals.** They stop AI from cheerfully suggesting things you already ruled out.
4. **Decisions explain *why*, not just *what*.** "We use Supabase" is weak. "We use Supabase because we needed row-level auth and didn't want to run our own Postgres — considered Firebase, rejected for vendor lock-in" is what makes the brain pay off six months later.
5. **Current State is a snapshot, not a journal.** Rewrite it freely. Old states aren't worth preserving.
6. **No invented facts.** If the user hasn't said it, write `(fill in)`. A brain with made-up details actively misleads future AI sessions — worse than no brain.

## File reference

- `references/templates.md` — The full templates for each of the six canonical files plus the README. Read this when creating a new brain.

## Quick start phrases the user might say

Map common requests to actions:

- "Start a project brain for <name>" → create a new brain folder with the six templates
- "List my brains" / "what brains do I have" → scan likely locations, show name + ST freshness + DC count
- "Compile my brain" → concatenate into a bundle and present it (ask which brain if ambiguous)
- "Log a decision: <text>" → prepend a DC entry (confirm which brain)
- "Update the state" → archive old ST to `.history/`, ask what changed, rewrite ST
- "What's changed?" / "diff my state" → compare current ST against most recent `.history/` snapshot
- "Add to glossary: <term>" → append to GL
- "What does my brain say about <topic>" → search the existing files and quote the relevant section
