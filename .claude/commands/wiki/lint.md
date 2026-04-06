---
name: wiki:lint
description: Health-check the wiki — finds orphan pages, contradictions, stale content, missing cross-references, and suggests next sources to investigate.
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - AskUserQuestion
---

<objective>
Health-check the wiki and keep it clean as it grows. Find structural problems (orphans, broken links, missing refs) and content problems (contradictions, stale facts, gaps). Optionally fix issues directly.
</objective>

<context>
Wiki root: current working directory ($PWD)
Schema: CLAUDE.md (in wiki root)
</context>

<process>

## Step 1: Discover areas and inventory all pages
Run `ls wiki/` to discover what wiki areas currently exist.
Use Glob to list all .md files under wiki/ (excluding index.md and overview.md).
Read wiki/index.md to compare — flag any pages on disk not in the index, or in the index but missing on disk.

## Step 2: Find orphan pages
For each wiki page, check whether any other page links to it using Grep.
Report pages with no inbound links as orphans. These are candidates for adding cross-references or for folding into related pages.

## Step 3: Find missing cross-references
Read all pages. When a page mentions something that has its own wiki page but doesn't link to it, flag it as a missing cross-reference.

## Step 4: Check for contradictions
Read all pages. Look for:
- Conflicting claims about the same entity or fact across pages
- Dated updates that contradict earlier content without noting the contradiction
- Status fields that seem stale relative to the content

## Step 5: Check for area gaps
When a concept, topic, or entity is mentioned frequently across pages but has no dedicated page of its own, flag it as a gap worth filling.

Also flag: content that exists in wiki/ but has no corresponding raw/ area, or raw/ areas with no wiki/ counterpart.

## Step 6: Suggest next sources
Based on the open questions and gaps found:
- Suggest 2-4 types of sources that would most improve the wiki
- Suggest 2-3 questions worth filing as query pages

## Step 7: Report
Produce a structured lint report:

```
## Wiki Lint Report — YYYY-MM-DD

### Orphan pages (N)
- [page](path) — no inbound links

### Missing cross-references (N)
- [page](path) mentions "[term]" but doesn't link to [term-page](path)

### Contradictions (N)
- [page-a](path) vs [page-b](path): both address [topic] differently

### Index gaps (N)
- On disk but not in index: ...
- In index but missing on disk: ...

### Content gaps (N)
- "[term]" appears on N pages but has no dedicated page

### Suggested sources
- ...

### Suggested queries
- ...
```

## Step 8: Fix offer
Ask: "Should I fix any of these issues now? I can: add cross-references, create missing pages, update the index, or note contradictions explicitly."

If yes, proceed with fixes, update index.md, then run: `git add -A && git commit -m "lint: fix cross-references and gaps"`

</process>
