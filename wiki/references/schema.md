# Wiki Schema & Templates

Conventions and copy-paste templates for the LLM Wiki skill. Read this before
writing your first page in a session. At Build time, a filled-in copy of the
`SCHEMA.md` template (bottom of this file) is written into the folder's `wiki/`
so the conventions travel with the wiki.

## Table of contents
1. Page types
2. Frontmatter spec
3. Naming conventions
4. Directory layout
5. Page templates (source, entity, concept, analysis)
6. Index template
7. Log template
8. Portable `SCHEMA.md` template (written into the folder)

---

## 1. Page types

| `type` | Purpose |
|---|---|
| `source` | A record of one ingested file/source and what it contained |
| `entity` | A concrete thing the wiki tracks (a person, account, project, device, dataset) |
| `concept` | An idea, topic, or theme that synthesises across sources |
| `analysis` | A synthesised comparison or conclusion derived from other pages |
| `index` | The map of a domain (or the whole wiki) |
| `log` | Append-only activity record for a domain |

## 2. Frontmatter spec

Every page begins with YAML frontmatter:

```yaml
---
type: source | entity | concept | analysis | index | log
title: "Human Readable Title"
description: "One-sentence summary of the page"
tags: [domain/<domain-name>, type/<page-type>]
sources: N          # number of distinct sources backing this page (0 for stubs)
updated: YYYY-MM-DD
---
```

`source` pages add one field:
```yaml
resource: "relative/path/from/root/to/the/original/file"
```

Rules:
- `type:` is mandatory and must be a top-level field, not only a tag.
- `tags` always includes a `domain/<name>` tag and a `type/<page-type>` tag.
- `updated` reflects the last meaningful edit, not a no-op touch.

## 3. Naming conventions

- Files are lowercase, hyphenated slugs: `quarterly-revenue.md`, not
  `Quarterly Revenue.md`.
- Source page slugs: `[origin]-[short-slug].md` where origin is the publication,
  author, or file source (e.g. `acme-pricing-2025.md`, `labs-2026-03.md`).
- One entity/concept per page; split rather than letting a page sprawl.

## 4. Directory layout

```
wiki/
├── SCHEMA.md              ← this wiki's conventions (filled-in template)
├── index.md               ← top-level map: links every domain index
└── <domain>/
    ├── index.md           ← domain map: Sources / Entities / Concepts / Analyses
    ├── log.md             ← append-only activity record
    ├── sources/           ← one page per ingested source
    │   └── <slug>.md
    ├── <entity>.md        ← entity pages live at the domain root
    ├── <concept>.md       ← concept pages live at the domain root
    └── <analysis>.md      ← analysis pages live at the domain root
```

## 5. Page templates

### Source page
```markdown
---
type: source
title: "<title>"
description: "<one-sentence summary>"
resource: "<relative/path/to/original>"
tags: [domain/<domain>, type/source]
sources: 1
updated: <YYYY-MM-DD>
---

# <title>

**Source:** `<relative/path>`  ·  **Ingested:** <YYYY-MM-DD>

## Summary
<2–5 sentence summary in your own words. Do not copy the source verbatim.>

## Key points
- <point> → relates to [[<entity-or-concept>]]
- <point>

## Notes / open questions
<anything unresolved, contradictory, or worth following up>
```

### Entity page
```markdown
---
type: entity
title: "<entity name>"
description: "<what this entity is>"
tags: [domain/<domain>, type/entity]
sources: <N>
updated: <YYYY-MM-DD>
---

# <entity name>

<concise description>

## Facts
- <attribute>: <value>  (from [[<source>]])

## Related
- [[<entity-or-concept>]]
```

### Concept page
```markdown
---
type: concept
title: "<concept>"
description: "<what this concept covers>"
tags: [domain/<domain>, type/concept]
sources: <N>
updated: <YYYY-MM-DD>
---

# <concept>

<synthesis across sources, in your own words>

## Backing sources
- [[<source>]]
```

### Analysis page
```markdown
---
type: analysis
title: "<analysis title>"
description: "<the question or comparison this answers>"
tags: [domain/<domain>, type/analysis]
sources: <N>
updated: <YYYY-MM-DD>
---

# <analysis title>

## Question
<what prompted this>

## Findings
<the synthesised conclusion, citing [[pages]]>

## Caveats / contradictions
<note disagreements between sources here>
```

## 6. Index template

One per domain; the top-level `wiki/index.md` follows the same shape but its
tables link to each domain's `index.md` instead of individual pages.

```markdown
---
type: index
title: "<Domain> Index"
description: "Map of the <domain> domain"
tags: [domain/<domain>, type/index]
sources: <N>
updated: <YYYY-MM-DD>
---

# <Domain> Index

## Sources
| Page | Description | Updated |
|---|---|---|
| [[sources/<slug>]] | <desc> | <date> |

## Entities
| Page | Description | Updated |
|---|---|---|
| [[<entity>]] | <desc> | <date> |

## Concepts
| Page | Description | Updated |
|---|---|---|
| [[<concept>]] | <desc> | <date> |

## Analyses
| Page | Description | Updated |
|---|---|---|
| [[<analysis>]] | <desc> | <date> |
```

## 7. Log template

```markdown
---
type: log
title: "<Domain> Log"
description: "Activity log for the <domain> domain"
tags: [domain/<domain>, type/log]
updated: <YYYY-MM-DD>
---

# <Domain> Log

## [<YYYY-MM-DD>] build | <domain>
Bootstrapped domain. Detected from: <files/folders>.

## [<YYYY-MM-DD>] ingest | <domain>
Processed <N> files: <list>. New pages: <list>. Updated: <list>.
```

## 8. Portable `SCHEMA.md` template (written into the folder at Build)

Fill in the domain list and write this to `[root]/wiki/SCHEMA.md` during Build,
so any future session — or any other LLM tool — can read the conventions
without this skill in context.

```markdown
# Wiki Schema

This wiki is LLM-maintained. Conventions:

- **Page types:** source, entity, concept, analysis, index, log.
- **Frontmatter:** every page has `type`, `title`, `description`, `tags`,
  `sources`, `updated`. Source pages also carry `resource` (path to the
  original file).
- **Cross-references:** use `[[wikilinks]]`.
- **Source files are immutable:** only files under `wiki/` are edited.
- **Every ingest updates the domain index and log.**

## Domains
<!-- filled in at build time -->
- **<domain>** — <one-line scope>

## Layout
- `wiki/index.md` — top-level map
- `wiki/<domain>/index.md` — domain map
- `wiki/<domain>/log.md` — activity log
- `wiki/<domain>/sources/` — one page per ingested source
- entity / concept / analysis pages live at the domain root
```
