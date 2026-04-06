---
name: wiki:query
description: Query the wiki — finds relevant pages, synthesizes an answer, and optionally files the result as a new page.
argument-hint: <question>
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
Answer a question using the accumulated knowledge in the wiki. The answer should synthesize across pages, cite sources, and surface connections that aren't obvious from any single page. Good answers can be filed back into the wiki — they are as valuable as ingested sources.
</objective>

<context>
Wiki root: current working directory ($PWD)
Schema: CLAUDE.md (in wiki root)
Question: $ARGUMENTS
</context>

<process>

## Step 1: Clarify the question
If $ARGUMENTS is empty, ask: "What would you like to know?"

If the question is ambiguous, ask one clarifying question before searching.

## Step 2: Discover areas and search the wiki
Run `ls wiki/` to see what areas exist.
Read wiki/index.md to identify all potentially relevant pages.

Read each relevant page. Be inclusive at this stage — a page that seems tangential may contain an important connection. Typical query reads 3-10 pages.

If a search term would help, use Grep to search across wiki/ for relevant keywords.

## Step 3: Synthesize the answer
Write a clear, structured answer that:
- Directly addresses the question
- Cites wiki pages (not raw sources) using markdown links
- Notes where wiki pages disagree or are uncertain
- Highlights non-obvious connections between areas

Use markdown formatting appropriate to the question type:
- Factual questions: prose with citations
- Comparisons: table
- How-to / process: numbered list
- Analysis: sections with headings

## Step 4: Surface gaps
After the answer, note in 1-3 bullets:
- What the wiki doesn't know yet that would improve this answer
- What sources would be most valuable to ingest next

## Step 5: Offer to file
Ask: "Should I file this answer as a wiki page in queries/?"

If yes:
- Create wiki/queries/<slug>.md with type: query frontmatter
- Sections: Question, Answer, Caveats, Date
- Update wiki/index.md
- Run: `git add -A && git commit -m "query: <question slug>"`

</process>
