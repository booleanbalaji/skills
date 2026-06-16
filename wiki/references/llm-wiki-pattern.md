# Reference: The LLM-wiki pattern

Background grounding for the `wiki` skill. This is the informal *practice* the
skill is built on. For the formal spec that standardizes it, see
`okf-spec.md`.

## What it is

The LLM-wiki pattern is the practice of maintaining a knowledge base as a
directory of plain markdown files that an AI agent reads before doing work and
updates as it learns. Humans curate the content and manage it like code (in
version control); the agent does the tedious upkeep.

It was articulated most crisply by Andrej Karpathy in his "LLM Wiki" gist
(widely shared, thousands of stars): https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f

## Why it works with LLMs

The chores that cause humans to abandon personal wikis — keeping
cross-references in sync, updating many files when one fact changes, never
forgetting to log a change — are exactly the kind of bookkeeping an LLM does
without tiring. An agent can touch many files in a single pass and keep the
graph consistent, which is what makes a living wiki sustainable where a manual
one decays.

## The recurring shape

The same pattern has reappeared under many names over the past year:
- Obsidian vaults wired to coding agents
- The `AGENTS.md` / `CLAUDE.md` family of convention files
- Repos full of `index.md` and `log.md` artifacts that agents consult before
  doing real work
- "Metadata as code" repositories inside data teams

They look alike — markdown, YAML frontmatter, cross-links — but each instance is
bespoke and not designed to interoperate. That gap is what OKF (see `okf-spec.md`)
exists to close.

## How this skill embodies the pattern

- A wiki is a directory of markdown files under `[root]/wiki/`.
- The agent reads `index.md` files to navigate before acting.
- Every change is recorded in an append-only `log.md`.
- Source files are immutable; only wiki pages are written.
- The wiki is meant to be version-controlled alongside whatever it documents.

It is a *practice*, not a *standard*: it has no conformance criteria of its own.
When portability and cross-tool sharing matter, conform to OKF.
