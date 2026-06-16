---
name: wiki
description: >
  Build and maintain an LLM-maintained wiki from the contents of whatever
  folder it is invoked in. Drop this skill into any folder and it reads the
  files, context, and structure already there and turns them into a linked
  Obsidian-style knowledge base. Use this skill for: building a wiki from a
  folder ("build a wiki here", "turn this folder into a wiki", "bootstrap a
  wiki for this project", "make a knowledge base from these files"); ingesting
  new files into an existing wiki ("ingest", "process this file", "add this to
  the wiki", "what's new in this folder"); querying it ("what does the wiki say
  about", "query the wiki", "summarise X from the wiki"); updating pages
  ("update the wiki", "file this as a page", "add a page for X"); and health
  checks ("lint the wiki", "what's stale", "find orphan pages"). ALWAYS invoke
  this skill before creating or editing any file under a folder's wiki/
  directory.
---

# Wiki Skill

A portable skill for turning any folder into an LLM-maintained knowledge base.
It is **folder-relative**: it makes no assumptions about location, domains, or
source paths. Everything it needs travels with the wiki it creates.

Throughout this skill, `[root]` means the folder the skill was invoked in (the
user's current working directory, or a folder they explicitly name). The wiki
lives at `[root]/wiki/`.

**Background (read once for grounding, not needed every session):** this skill
implements two established conventions. The structure and operations follow the
**LLM-wiki pattern** (`references/llm-wiki-pattern.md`), and the page model is
aligned with Google's **Open Knowledge Format (OKF) v0.1**
(`references/okf-spec.md`) — a directory of markdown concept files with YAML
frontmatter, where `type` is the only required field. Consult those references
when you need the rationale behind a convention, when deciding whether a wiki
should be OKF-conformant for sharing, or when the user asks about either.

---

## Step 0 — Orient, then detect the operation

Before doing any work:

1. Confirm `[root]` — the folder you are operating in. If the user named a
   folder, use it; otherwise use the current working directory.
2. Check whether `[root]/wiki/` already exists.
   - **No `wiki/`** → this is almost certainly a **Build** (first run).
   - **`wiki/` exists** → read `[root]/wiki/SCHEMA.md` to load the conventions
     this wiki was built with, then proceed with the requested operation.

Then identify the operation:

| User says | Operation |
|---|---|
| "build a wiki", "turn this folder into a wiki", "bootstrap", "make a knowledge base" | → **Build** |
| "ingest", "process this", "add this to the wiki", "what's new in this folder" | → **Ingest** |
| "what does the wiki say", "query", "summarise", a question about wiki content | → **Query** |
| "update the wiki", "file this as a page", "add a page for X" | → **Update** |
| "lint", "health check", "what's stale", "orphan pages" | → **Lint** |

If `wiki/` does not exist and the user asks for anything other than Build, offer
to Build first — there is nothing to query, update, or lint yet.

If intent is ambiguous, ask one question before proceeding.

The full frontmatter spec, page templates, and index/log templates live in
`references/schema.md`. Read it before writing your first page in a session.

---

## BUILD — Bootstrap a wiki from the folder's contents

This is the core operation and the reason the skill exists. The goal: read what
is already in `[root]` and infer a sensible wiki structure, **with the user
confirming the structure before anything is written**.

### 1. Scan the folder

Walk `[root]` (skip `wiki/` itself, hidden files, and dependency/build
directories like `node_modules`, `.git`, `venv`). For each file collect cheap
signals only — do not read entire large files yet:
- Relative path and the subfolder it sits in
- File type / extension
- A content sample: for text/markdown, the first ~200 characters plus any top
  headings; for PDFs, the title/first page; for CSVs, the header row; for code,
  the file's docstring or top comment

### 2. Detect domains

Cluster the files into candidate **domains** (top-level sections of the wiki).
Signals, in priority order:
1. **Existing subfolders** are strong domain candidates — a folder named
   `finance/` or `research/` or `contracts/` is very likely its own domain.
2. **Content clusters** — files that share vocabulary, topic, or type group
   together even when loose in the root.
3. **File-type clusters** — e.g. a pile of lab PDFs vs. a pile of invoices.

Aim for a small number of meaningful domains (typically 2–6). Avoid one domain
per file. A single coherent folder may legitimately be **one** flat domain.

### 3. Handle thin or ambiguous folders

If there is too little signal to infer domains confidently (very few files, or
no discernible clusters), **do not guess**. Ask the user whether to:
- proceed as a single flat wiki, or
- name the domains themselves.

Default suggestion in this case: **ask**.

### 4. Confirm before scaffolding

Present the detected structure to the user and **wait for approval**:
- The list of proposed domains
- Which files map to which domain
- Anything that didn't classify cleanly (list it; ask where it goes or whether
  to leave it unfiled)

Do not create any files until the user confirms or adjusts.

### 5. Scaffold

Once confirmed, create:
```
[root]/wiki/
├── SCHEMA.md          ← conventions for THIS wiki (from references/schema.md)
├── index.md           ← top-level map linking every domain index
└── [domain]/
    ├── index.md       ← Sources, Entities, Concepts, Analyses tables
    └── log.md         ← append-only activity record
```
Write `wiki/SCHEMA.md` first (copy the template from `references/schema.md` and
fill in the confirmed domain list) so the conventions are portable and future
sessions stay consistent. Populate initial entity/concept stubs for anything the
scan already made obvious. Then run an initial **Ingest** pass over the scanned
files (next section) so the wiki has real content, not just empty scaffolding.

Log the build event in each domain's `log.md`.

---

## INGEST — Process files into the wiki

Use to fold new (or not-yet-processed) files in `[root]` into the wiki. This is
run automatically at the end of Build, and on demand afterwards.

### Find what's new
A file is already ingested if a source page references it under
`wiki/[domain]/sources/`, or it appears in a domain `log.md`. Compare the
folder's files against those records; anything unaccounted for is a candidate.

If the user only wants to *see* what's new without committing (a dry run),
present the candidate list grouped by likely domain and stop there.

### Route to a domain
Use the domains defined in `wiki/SCHEMA.md`. Route by, in order: (1) the
subfolder the file lives in, (2) content match to an existing domain, (3) file
type. If a file fits no existing domain, surface it — propose a new domain or
ask — rather than forcing a bad fit.

### Process by type
- **Markdown / text** — short items (links, notes) append to a collector page
  (e.g. `sources/notes.md`); substantive pieces get their own source page.
- **PDF** — extract text (pdfplumber or the Read tool); create a source page and
  update any entity/concept pages the content touches. **If the PDF is scanned or
  image-heavy** (extraction returns little/no text, or the content lives in
  figures, charts, and tables), do not record a near-empty page — escalate (see
  "Heavier extraction" below).
- **CSV / tabular** — identify the dataset from its headers; create or update
  the relevant data/entity page rather than dumping raw rows.
- **Office / rich / binary formats** (`.docx`, `.pptx`, `.xlsx`, `.html`,
  images) — convert to markdown with a local tool before ingesting so tables and
  structure survive (see "Heavier extraction").
- **Code / configs** — summarise purpose and key interfaces into a source or
  concept page; link back to the file. Never copy whole files in verbatim.

### Heavier extraction (when simple text extraction is insufficient)

Some files won't yield their content to a plain text read — scanned/image-heavy
PDFs, Office docs, slide decks, spreadsheets, HTML, and standalone images. Reach
for a more capable **local** tool rather than ingesting a thin or empty page:

- **MarkItDown** (`pip install markitdown`, or `markitdown <file>`) converts
  PDF, Office formats, HTML, and images to structured markdown — a good default
  first escalation for most rich formats.
- **OCR** for scanned/image PDFs: `ocrmypdf` to add a text layer, or
  `pytesseract` on rasterized pages.
- **Vision fallback**: if a page is genuinely visual (diagrams, charts,
  handwritten notes), rasterize it and read it with a vision-capable model, then
  write a prose/structured summary of what it shows.

Confirm the tool is available before relying on it; if it isn't installed, tell
the user the install command and what it unlocks rather than silently degrading.
**Thin-extraction guardrail:** if a file yields suspiciously little text for its
size or page count, assume the simple extractor failed and escalate before
writing the source page — never log a near-empty page as a successful ingest.

### Write wiki content
For every source ingested, write at minimum:
1. A source page `wiki/[domain]/sources/[slug].md` with full frontmatter
2. Updates to the entity/concept pages it affects (check the domain index first)
3. A row in `wiki/[domain]/index.md` (Sources table)
4. An entry in `wiki/[domain]/log.md`

**Before creating any page, read `wiki/[domain]/index.md`** to check for an
equivalent page. Updating beats duplicating.

### After ingesting
Report: how many files processed, into which domains, key new findings, and any
contradictions found with existing wiki content.

---

## QUERY — Answer questions from the wiki

1. Identify the likely domain(s) from the question.
2. Read `wiki/index.md` and the relevant `wiki/[domain]/index.md` to locate
   pages.
3. Read those pages, following `[[wikilinks]]` as needed.
4. Answer clearly, citing the pages used with inline `[[wikilinks]]`.
5. If the answer is substantive (synthesis, comparison, analysis), offer:
   *"Want me to file this as a wiki page?"* If yes, write it, update index + log.

Anything worth synthesising is worth preserving — good answers compound the
knowledge base.

---

## UPDATE — Add or modify pages directly

Use when the user has structured information to record (a decision, a change, a
new fact) rather than a raw file to ingest.

1. Decide whether existing pages should be updated or a new page is needed.
2. Read the relevant existing pages first.
3. Write the change. Never edit the original source files in `[root]` — only
   pages under `wiki/`.
4. Update index + log.

**Check-before-create:** if unsure a page exists, read the domain `index.md`
first. Prefer updating over near-duplicating.

---

## LINT — Wiki health check

Run on a domain or the whole wiki.

1. **Orphan pages** — pages not linked from any other wiki page.
2. **Stale claims** — pages built from data old enough that newer sources in the
   folder likely supersede them.
3. **Missing cross-references** — concepts mentioned but lacking their own page.
4. **Contradictions** — inconsistent facts between pages (flag, don't resolve).
5. **Unfiled files** — files in `[root]` not yet ingested.
6. **Suggest** 2–3 questions or sources worth investigating next.

Present a report. Do **not** auto-fix — flag and wait for direction. Append
`## [YYYY-MM-DD] lint | [domain]` to the relevant log.

---

## Key rules (every operation)

- **Source files are immutable.** Never modify the original files in `[root]`;
  only ever write under `[root]/wiki/`.
- **Read the index before creating.** Always check for an existing page first.
  Duplicate pages dilute the knowledge base.
- **Every ingest updates index + log.** No exceptions.
- **Cross-references use `[[wikilinks]]`** so Obsidian's graph view works.
- **When sources contradict**, note it explicitly on the relevant pages. Never
  silently pick a winner.
- **Confirm structure before scaffolding.** Build never creates a directory tree
  the user hasn't approved.
- **Type on every page.** All pages carry a `type:` frontmatter field, not just
  a tag (see `references/schema.md`).

---

## Maintaining this skill (optional, Claude Code only)

When iterating on this skill, you can optionally tune its trigger `description`
with skill-creator's description optimizer:

```bash
python -m scripts.run_loop \
  --eval-set <trigger-eval.json> \
  --skill-path <path-to-this-skill> \
  --model <model-id> --max-iterations 5 --verbose
```

It splits an eval set into train/held-out, measures how reliably the description
triggers, proposes improvements, and returns a `best_description` to paste into
the frontmatter. This step requires the `claude` CLI, so it is **only available
in Claude Code** (skip it in web chat). It is a maintenance task, not part of
normal wiki use.

---

## Quick reference

| Thing | Where |
|---|---|
| Wiki root | `[root]/wiki/` |
| This wiki's conventions | `[root]/wiki/SCHEMA.md` (written at Build) |
| Top-level map | `[root]/wiki/index.md` |
| Per-domain pages | `[root]/wiki/[domain]/` |
| Frontmatter spec + templates | `references/schema.md` (in this skill) |
| LLM-wiki pattern (rationale) | `references/llm-wiki-pattern.md` |
| OKF v0.1 spec (alignment) | `references/okf-spec.md` |
