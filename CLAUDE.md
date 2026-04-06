# Wiki Schema

This is the schema for this user's personal wiki. You (the LLM) own the `wiki/` layer entirely — you create pages, update them, maintain cross-references, and keep everything consistent. The user curates sources and asks questions. You do the bookkeeping.

## Directory layout

```
wiki/
├── CLAUDE.md     ← this file (schema)
├── raw/          ← source documents (read-only, never modify)
│   └── <area>/   ← one subdirectory per input area (discover with: ls raw/)
└── wiki/         ← LLM-maintained wiki (you own this)
    ├── index.md  ← master catalog (update on every change)
    ├── overview.md  ← high-level synthesis of everything
    └── <area>/   ← one subdirectory per wiki area (discover with: ls wiki/)
```

**Areas are open-ended.** Do not assume a fixed list. At the start of any operation, run `ls raw/` and `ls wiki/` to discover what areas currently exist. New areas are created by simply making a new subdirectory — no schema change required.

Two special files always exist at the wiki root level: `wiki/index.md` and `wiki/overview.md`. Everything else is in area subdirectories.

**Projects with their own codebase** live inside `wiki/` as a regular area directory (e.g. `wiki/crupp-de/`). Their `index.md` is the wiki page. Build tooling lives in `cmd/` alongside it. Links from such an `index.md` to other wiki pages use normal relative paths (e.g. `../career/overview.md`).

`raw/clippings/` is the conventional home for web articles fetched by URL.

## Page frontmatter

Every wiki page must have YAML frontmatter:

```yaml
---
type: source|concept|entity|project|investment|note|query
tags: [tag1, tag2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: [raw/path/to/source.md]   # for source pages; for others, list sources that inform this page
---
```

Use `tags` consistently. Common tags:
- `finance`, `startup`, `technology`, `health`, `psychology`, `investing`
- Entity types: `person`, `company`, `fund`, `market`, `sector`
- Status: `active`, `archived`, `speculative`, `confirmed`

## Page types and conventions

### Source pages (`wiki/sources/`)
Filename: `YYYYMMDD-<area>-<slug>.md` — date ingested, raw area (e.g. `clippings`, `investments`, `recipes`), then slug from title. Example: `20260406-clippings-massaman-curry.md`, `20260406-investments-kapitalanlagen.md`

Required sections:
- **Summary** — 2-4 sentence summary of what the source says
- **Key points** — bulleted takeaways
- **Connections** — links to related wiki pages this source informs or updates
- **Raw source** — link back to the raw file

Keep these factual and neutral. Your job is to extract, not editorialize.

### Concept pages (`wiki/concepts/`)
Filename: the concept name, e.g. `network-effects.md`, `dollar-cost-averaging.md`

Required sections:
- **Definition** — concise definition
- **How it works** — mechanism, not just label
- **Examples** — at least one concrete example from the sources
- **Related concepts** — links
- **Sources** — what informed this page

Update concept pages when new sources add nuance, examples, or contradictions.

### Entity pages (`wiki/entities/`)
Filename: entity name, e.g. `sequoia-capital.md`, `sam-altman.md`

Required sections (as applicable):
- **About** — who/what they are
- **Relevance** — why they appear in the wiki
- **Key facts** — important data points
- **Relationships** — links to related entities and projects
- **Sources** — what informed this page

### Incubator pages (`wiki/incubator/`)
Active and in-progress projects. Each lives in its own subdirectory: `wiki/incubator/<name>/`. The main file is always `index.md`. Additional files (roadmap, notes, decisions, research, etc.) can be added alongside it as the project grows.

Directory name: project slug, e.g. `wiki/incubator/upscaledb/`, `wiki/incubator/crupp-de/`

`index.md` required sections:
- **Idea summary** — one paragraph pitch
- **Problem** — what problem it solves
- **Market** — who would pay, how big
- **Differentiation** — why this, why now
- **Open questions** — unresolved issues
- **Status** — `idea | exploring | validating | building | paused | killed`
- **Related** — links to concepts, entities, investments that inform this
- **Sources** — research and notes that inform this

When creating a new project: `mkdir wiki/incubator/<name>/` and create `index.md` inside it.
When looking up an existing project: look for `wiki/incubator/<name>/index.md`.

### Archive pages (`wiki/archive/`)
Finished projects — shipped, killed, handed off, or otherwise concluded. Same subdirectory structure as incubator: `wiki/archive/<name>/index.md`, with any additional files alongside it.

To archive a project: `git mv wiki/incubator/<name> wiki/archive/<name>`, then add an **Outcome** section to `index.md` documenting what happened and why it concluded.

`index.md` additions vs incubator:
- **Outcome** — what happened: shipped / killed / handed off / superseded. Include date and a brief reason.
- **Status** should be updated to `shipped | killed | handed-off | superseded`

Update `wiki/index.md` to move the entry from the Incubator section to the Archive section.

### Investment pages (`wiki/investments/`)
Filename: asset or position name, e.g. `bitcoin.md`, `nvidia.md`, `eur-usd.md`

Required sections:
- **Thesis** — why this investment makes sense
- **Position** — what's held (or being tracked), entry/exit thinking
- **Key facts** — relevant metrics, catalysts
- **Risks** — what could invalidate the thesis
- **Updates** — dated bullets when thesis or facts change
- **Sources** — research informing this

### Note pages (`wiki/notes/`)
Filename: slug from title, e.g. `thoughts-on-compounding.md`

Free-form, but must have frontmatter and link to related pages. These are zettelkasten atoms — one idea per note, densely cross-referenced.

### Query pages (`wiki/queries/`)
Filename: slug from question, e.g. `which-markets-benefit-from-ai.md`

Required sections:
- **Question** — the original question
- **Answer** — the synthesized answer with citations to wiki pages
- **Caveats** — what's uncertain
- **Date** — when this was answered

## index.md format

`wiki/index.md` is a catalog updated on every change. It has one section per wiki area (discovered from `ls wiki/`). Structure:

```markdown
# Wiki Index

_Last updated: YYYY-MM-DD. N pages total._

## <Area> (N)
- [Title](<area>/filename.md) — one-line summary or description
```

One section per area subdirectory, in whatever order makes sense. Keep each entry to one line. Do not add prose between sections. Add new area sections as new areas are created.

## Cross-referencing rules

1. When you create or update a page, update **all related pages** that should link to it.
2. Every page should have at least one inbound link (except very new pages).
3. Use relative markdown links: `[Network effects](../concepts/network-effects.md)`
4. When a new source contradicts an existing page, **note the contradiction explicitly** on the relevant wiki page with a dated bullet.
5. Update `overview.md` when something materially changes the big picture.

## Operations

### Ingest workflow
When processing a new source (`/wiki:ingest`):
1. Read the raw source file
2. Identify: what type of source is this? what areas does it touch?
3. Create a summary page in `wiki/sources/`
4. Identify all wiki pages that should be updated (concept, entity, project, investment, note pages)
5. Create any pages that don't yet exist
6. Update all affected pages — add new facts, note contradictions, strengthen links
7. Update `wiki/index.md` (add new pages, update counts)
8. Summarize what was done
9. **Commit:** `git add -A && git commit -m "ingest: <source title>"`

### Query workflow
When answering a question (`/wiki:query`):
1. Read `wiki/index.md` to find relevant pages
2. Read all relevant pages
3. Synthesize an answer with citations (link to wiki pages, not raw sources)
4. Ask whether to file the answer — if yes, create a page in `wiki/queries/`
5. If filed, update `wiki/index.md`
6. **Commit (if filed):** `git add -A && git commit -m "query: <question slug>"`

### Lint workflow
When health-checking the wiki (`/wiki:lint`):
1. Read all pages
2. Report: orphan pages (no inbound links), contradictions, stale facts, missing cross-refs, important concepts without their own page
3. Suggest new sources to seek or questions to investigate
4. Optionally fix issues directly
5. **Commit (if fixes applied):** `git add -A && git commit -m "lint: fix cross-references and gaps"`

### New note workflow
When creating a note (`/wiki:new-note`):
1. Create a focused, atomic note in `wiki/notes/`
2. Identify related existing pages and add cross-references
3. Update `wiki/index.md`
4. **Commit:** `git add -A && git commit -m "note: <title>"`

## Conventions

- Dates: always ISO 8601 (`YYYY-MM-DD`)
- Filenames: lowercase, hyphens, no spaces (`my-page-title.md`)
- No orphan pages: always link new pages from at least one existing page
- Update `updated:` frontmatter whenever you change a page
- **Git commits:** after every major update (ingest, filed query, note, lint fixes), run `git add -A && git commit -m "<type>: <description>"`. Types: `ingest`, `query`, `note`, `lint`, `update`
- When in doubt about where something belongs, put it in `wiki/notes/` and cross-reference aggressively
- **`cmd/` convention:** when a project has an associated command (build, publish, deploy, generate, etc.), create a `cmd/` subdirectory inside the project directory. `cmd/Makefile` is required and must define at least two targets: `build` and `deploy`. The wiki page (`index.md`) stays at the project root, not inside `cmd/`. Example: `wiki/crupp-de/index.md` + `wiki/crupp-de/cmd/Makefile`.
