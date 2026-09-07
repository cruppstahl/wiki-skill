---
name: wiki:build-kb
description: Build or rebuild a knowledge base from a directory of source files. Processes multiple sources at once, deduplicates, resolves conflicts, and writes dense LLM-optimized topic files into a two-level KB structure.
argument-hint: <raw-source-directory> [wiki-area]
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
Transform a directory of raw source files into a structured, dense, LLM-optimized knowledge base. The output is a two-level hierarchy: a routing index and per-topic files using fact-list format with explicit conflict resolution. Sources are processed once, deduplicated, and synthesized — not summarized one by one.
</objective>

<context>
Wiki root: current working directory ($PWD)
Arguments: first token is the source directory (e.g. raw/fitness/); optional second token is the wiki area (e.g. fitness). If area is omitted, infer from the source directory name.
</context>

<process>

## Step 1: Validate input
If no source directory is provided, ask: "Which raw/ directory should I build the KB from?"

Resolve the wiki area: if not provided, use the last segment of the source path (e.g. `raw/fitness/` → `fitness`).

Check that the source directory exists and identify pending vs already-processed files:
```bash
find <source-directory> -maxdepth 1 -type f | sort
```

**Resume detection:** a file that is 0 bytes has already been processed in a previous run. Separate the file list into:
- **Done** (size = 0): skip entirely — do not read, do not re-process
- **Pending** (size > 0): process these

Report to the user: "Found N files total: X already processed (skipped), Y remaining."

If all files are 0 bytes, the KB is complete — report and stop.

## Step 2: Scan all sources
Read every **pending** (non-empty) file in the source directory. For large sets (>10 files), read in batches and take notes between batches — do not try to hold all content in memory at once.

**PDFs:** use the Read tool with `pages: "1-20"` per call (max 20 pages). For PDFs longer than 20 pages, read in chunks (1-20, 21-40, …) and accumulate before moving on.

While reading, build a mental map:
- What topics appear across sources?
- Which claims appear in multiple sources (consensus candidates)?
- Where do sources disagree (conflict candidates)?
- What is the approximate coverage of each topic?

## Step 3: Determine KB structure
Check if `wiki/<area>/kb/index.md` already exists.

**If KB exists:** read `wiki/<area>/kb/index.md` and the existing topic files. You will be merging into existing content, not starting from scratch.

**If KB does not exist:** propose a topic breakdown to the user based on Step 2 findings:
- List the proposed topic files with one-sentence scope descriptions
- Ask: "Does this topic breakdown look right? Anything to add, merge, or rename?"
- After confirmation, create the scaffold: `wiki/<area>/kb/` directory, `index.md`, and one file per topic using the KB scaffold format (see CLAUDE.md KB conventions).

## Step 4: Process topic by topic
For each topic file, work through all sources and extract only what is relevant to that topic.

Write in fact-list format — not prose. Each section contains:
- Specific, actionable claims
- Numeric targets where sources provide them
- Conflict markers where sources disagree
- Resolution calls where you can make them

**Fact-list format:**
```
## <Section heading>
- CONSENSUS: <claim supported by multiple sources>
- <claim from single source, stated plainly>
- CONFLICT [source-A vs source-B]: source-A claims X; source-B claims Y
  RESOLVED: <your synthesis> — rationale: <brief reason>
```

Rules:
- Do not repeat a claim already captured in the same file, even if multiple sources say it
- Do not use prose transitions ("it is important to note that", "additionally", etc.)
- Numeric specificity beats vagueness: "10–20 sets/week" beats "moderate volume"
- When sources conflict on numbers, state both ranges and note which has stronger evidence if discernible

## Step 5: Update kb/index.md
For each topic file, write or rewrite its routing paragraph in `kb/index.md`:
- 2-4 sentences summarising what the topic file covers
- Specific enough that a query can decide relevance without opening the file
- Note any major conflicts present in the topic file

## Step 6: Create source summary pages and mark as done
For each source file processed, in order:
1. Create a summary page in `wiki/sources/` (if one does not already exist) using the standard source page format. The source page is the audit trail; the KB topic files are the synthesised knowledge.
2. Immediately truncate the source file to 0 bytes to mark it as done:
   ```bash
   truncate -s 0 <source-file-path>
   ```
   Do this **per file, right after its summary page is written** — not at the end of the batch. This ensures that if the session ends mid-run, already-processed files are not re-processed on resume.
Do not commit during the import — the single final commit in Step 8 covers everything.

## Step 7: Update wiki/index.md
- Add or update the KB entry lines in `wiki/index.md` under the area section
- Use the rich entry format: `[Title](path) — summary; keywords: kw1, kw2, kw3`
- Update the page count and "Last updated" date

## Step 8: Commit
```bash
git add -A && git commit -m "build-kb: <area> — <N> sources, <M> topics"
```

## Step 9: Report
Summarise:
- How many sources were processed
- How many topic files were created or updated
- How many CONFLICT markers were added and how many were RESOLVED
- Any topics that need more sources to be well-covered (gaps)

</process>
