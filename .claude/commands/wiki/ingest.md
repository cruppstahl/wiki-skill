---
name: wiki:ingest
description: Ingest a source into the wiki — a local file or a URL. Reads/fetches it, extracts key information, and integrates it into the existing wiki pages.
argument-hint: <path-to-file or URL>
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - WebFetch
  - AskUserQuestion
---

<objective>
Ingest a raw source into the personal wiki. The goal is not just to summarize the source, but to integrate it — updating existing pages, creating new ones, and strengthening cross-references so the wiki compounds with every new source.
</objective>

<context>
Wiki root: current working directory ($PWD)
Schema: CLAUDE.md (in wiki root)
Source argument: $ARGUMENTS
</context>

<process>

## Step 1: Read the schema
Read CLAUDE.md to understand conventions, page formats, and frontmatter requirements.

## Step 2: Discover areas
Run `ls raw/` to see what input areas exist.
Run `ls wiki/` to see what wiki areas exist.
These are the live areas — do not assume a fixed list.

## Step 3: Identify and fetch the source
If $ARGUMENTS is not provided, ask: "What should I ingest? Provide a file path (relative to raw/) or a URL."

**Filename convention for all stored source files:**
`YYYYMMDD-<area>-<slug>.md` where date is today, area is the raw/ subdirectory (e.g. `clippings`, `investments`, `recipes`), and slug is derived from the title or filename. Example: `20260406-clippings-massaman-curry.md`

**If $ARGUMENTS is a URL** (starts with http:// or https://):
- Use WebFetch to retrieve the page content
- Save the fetched content as `raw/clippings/YYYYMMDD-clippings-<slug>.md`
- The raw/clippings/ file is the permanent source record

**If $ARGUMENTS is a local file path:**
- Read the file fully (treat as relative to the wiki root if not absolute)
- The area is the raw/ subdirectory the file lives in (e.g. `raw/investments/` → area is `investments`)

## Step 4: Assess the source
Determine:
- What type of content is this? (article, note, report, data file, personal writing, etc.)
- Which wiki areas does it touch? (check against areas discovered in Step 2)
- What are the 3-5 most important takeaways?
- What existing wiki pages might this update?

Read wiki/index.md to see what pages already exist.

## Step 5: Discuss with the user (brief)
Share your assessment in 3-4 sentences: what the source is about and what you plan to do with it. Ask if there's anything specific to emphasize or de-emphasize before you proceed.

## Step 6: Create the source summary page
Create `wiki/sources/YYYYMMDD-<area>-<slug>.md` following the source page format from CLAUDE.md (same date, area, and slug as the raw file):
- type: source
- Summary, Key points, Connections, Raw source sections
- Frontmatter with tags, created date, sources field

## Step 7: Update the wiki
For each wiki area the source touches:
- If a relevant page exists: read it, add new facts/examples/contradictions, update the `updated:` frontmatter date
- If no relevant page exists: create it in the appropriate area directory, following the format from CLAUDE.md
- If the content belongs to an area that doesn't exist yet: create the directory and a new page

**Incubator pages are subdirectories:** each active project lives at `wiki/incubator/<name>/index.md`. When creating a new project page, first `mkdir wiki/incubator/<name>/`, then create `index.md` inside it. When looking up an existing project, check both `wiki/incubator/<name>/index.md` and `wiki/archive/<name>/index.md` (it may have been archived).

A single source should typically touch 3-10 wiki pages. Be thorough — this is the key value of the system.

When a source contradicts an existing claim, note it explicitly:
> **Contradiction [YYYY-MM-DD]:** [source] claims X, but [earlier-source] claimed Y. See [source-page](../sources/...).

## Step 8: Update index.md
Read wiki/index.md, add all new pages, update counts and the "Last updated" date.

## Step 9: Commit
Run: `git add -A && git commit -m "ingest: <source title>"`

## Step 10: Report
Summarize what was done: how many pages created, how many updated, any notable connections or contradictions surfaced.

</process>
