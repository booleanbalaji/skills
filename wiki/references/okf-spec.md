# Reference: Open Knowledge Format (OKF) v0.1

Background grounding for the `wiki` skill. OKF is the formal specification that
standardizes the LLM-wiki pattern (see `llm-wiki-pattern.md`). Details below are
drawn from the v0.1 draft spec.

- Spec, sample bundles, reference implementations (Apache 2.0):
  https://github.com/GoogleCloudPlatform/knowledge-catalog/tree/main/okf
  (spec file: `okf/SPEC.md`)
- Announcement (Google Cloud, 12 June 2026; Sam McVeety & Amir Hormati):
  https://cloud.google.com/blog/products/data-analytics/how-the-open-knowledge-format-can-improve-data-sharing

OKF is a *format*, not a service or platform — no schema registry, no central
authority, no required SDK or runtime. If you can read a file you can read OKF;
if you can clone a repo you can ship it. It is minimally opinionated: it
standardizes only the structural conventions needed to make a knowledge corpus
self-describing and leaves everything else to the producer.

## Terminology

- **Knowledge Bundle** — a self-contained, hierarchical directory of knowledge
  documents; the unit of distribution.
- **Concept** — one unit of knowledge, represented as a single markdown file.
  May describe a tangible asset (a table, an API) or an abstract idea (a metric,
  a process).
- **Concept ID** — the concept file's path within the bundle with `.md` removed
  (`tables/users.md` → `tables/users`).
- **Frontmatter / Body** — the YAML block at the top, and everything after it.
- **Link** — a standard markdown link between concepts. **Citation** — a link to
  an external source backing a claim.

## Bundle structure

A bundle is a directory tree of markdown files; the layout is domain-independent.
It MAY be distributed as a git repo (recommended — history, attribution, diffs),
a tarball/zip, or a subdirectory of a larger repo.

### Reserved filenames

Only two filenames are reserved (defined meaning at any level; MUST NOT be used
for concept documents):

| Filename | Purpose |
|---|---|
| `index.md` | Directory listing for progressive disclosure (§ Index files) |
| `log.md` | Chronological update history (§ Log files) |

Every other `.md` file is a concept document.

## Concept documents

Each concept is a UTF-8 markdown file: a YAML frontmatter block delimited by
`---` lines, then a free-form markdown body.

### Frontmatter fields

- **Required:** `type` — a short, non-empty string identifying the kind of
  concept (e.g. `BigQuery Table`, `Metric`, `Playbook`, `Reference`). Type
  values are **not** registered centrally; producers pick descriptive values and
  consumers MUST tolerate unknown types (treat as generic).
- **Recommended (priority order):** `title`, `description` (one sentence),
  `resource` (canonical URI of the underlying asset; absent for abstract
  concepts), `tags` (list of short strings), `timestamp` (ISO 8601 last-modified).
- **Extensions:** producers MAY add any keys; consumers SHOULD preserve unknown
  keys and MUST NOT reject documents that carry them.

### Body

Standard markdown, favoring structure (headings, lists, tables, code fences)
over prose. No required sections. Conventional headings, used when applicable:
`# Schema` (an asset's columns/fields), `# Examples`, `# Citations`.

## Cross-linking

Concepts link via standard markdown links. Two forms:
- **Absolute / bundle-relative** — begin with `/`, resolved from the bundle root
  (`[customers](/tables/customers.md)`). **Recommended**: stable when files move.
- **Relative** — ordinary relative paths (`[x](./other.md)`).

The kind of relationship is conveyed by surrounding prose, not the link; graph
consumers treat links as untyped directed edges. Consumers MUST tolerate broken
links — a missing target may just be not-yet-written knowledge.

## Index files

`index.md` MAY appear in any directory (including root) for progressive
disclosure. It carries **no frontmatter** (the sole exception: a bundle-root
`index.md` MAY declare `okf_version: "0.1"`). Body is one or more heading-grouped
sections of bulleted links with short descriptions, e.g.
`* [Title](relative-url) - description`. Producers MAY generate it; consumers MAY
synthesize one when absent.

## Log files

`log.md` MAY appear at any level to record that scope's change history: a flat
list of date-grouped entries, newest first. Date headings MUST be ISO 8601
`YYYY-MM-DD`. A leading bold verb (`**Update**`, `**Creation**`,
`**Deprecation**`) is convention, not a requirement.

## Citations

Claims sourced from external material SHOULD be listed under a numbered
`# Citations` heading. Citation links MAY be absolute URLs, bundle-relative
paths, or paths into a `references/` subdirectory that mirrors external sources
as first-class concepts.

## Conformance (v0.1)

A bundle conforms if:
1. Every non-reserved `.md` file has a parseable YAML frontmatter block.
2. Every such block has a non-empty `type`.
3. Any `index.md` / `log.md` present follows its defined structure.

Everything else is soft guidance. Consumers MUST NOT reject a bundle for missing
optional fields, unknown `type` values, unknown extra keys, broken cross-links,
or missing `index.md`. This permissive model is intentional, so bundles stay
usable as they grow and get partially agent-generated.

## Versioning

Versions are `<major>.<minor>`; minor bumps are backward-compatible additions,
major bumps may break. A bundle MAY declare its target version via
`okf_version: "0.1"` in the root `index.md` frontmatter. Unknown versions →
best-effort consumption, not rejection.

## How this skill relates to OKF

This skill targets OKF-aligned output: a directory of markdown concept files,
`type` on every concept, cross-linking into a graph, optional `index.md` /
`log.md`. It diverges from strict v0.1 conformance in ways chosen for this
skill (an Obsidian-first personal vault), **not** required by OKF:

1. **Links** — uses Obsidian `[[wikilinks]]`; OKF requires standard markdown
   links (bundle-relative `/path.md` preferred).
2. **Timestamp** — templates use `updated:`; OKF uses `timestamp:`.
3. **Reserved-file frontmatter** — this skill puts `type: index` / `type: log`
   frontmatter on `index.md` / `log.md`; under OKF those are reserved files that
   carry no frontmatter (except an optional `okf_version` in the root index).
4. **Type vocabulary** — this skill uses a fixed set
   (`source/entity/concept/analysis`); OKF treats `type` as a free string.

For a personal Obsidian vault these are fine. To emit a *conformant* OKF bundle
(for sharing or cross-tool consumption), switch to bundle-relative markdown
links, rename `updated:` → `timestamp:`, drop frontmatter from `index.md` /
`log.md`, and allow free-form `type` values.
