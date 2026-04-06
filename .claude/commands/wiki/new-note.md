---
name: wiki:new-note
description: Create a new zettelkasten-style note in the wiki, with cross-references to related pages.
argument-hint: <title or idea>
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
Create a focused, atomic note and integrate it into the wiki. A good note captures one idea, links to related pages, and enriches the graph — not just a file dump.
</objective>

<context>
Wiki root: current working directory ($PWD)
Schema: CLAUDE.md (in wiki root)
Title/idea: $ARGUMENTS
</context>

<process>

## Step 1: Capture the idea
If $ARGUMENTS is provided, use it as the note title/seed. If not, ask: "What's the note about?"

If the user has more to say about the content, ask: "What do you want to capture? (You can paste text, bullet points, or just describe the idea.)"

## Step 2: Discover existing areas
Run `ls wiki/` to see what wiki areas currently exist.
Read wiki/index.md to see what pages are already there.

## Step 3: Determine the right area
Based on the content and the discovered areas, decide where this belongs:
- If it's a focused personal reflection or fleeting idea → wiki/notes/
- If it clearly matches another existing area (e.g. wiki/recipes/, wiki/incubator/) → use that area instead

If it fits better in another area, say so and create the right page type there.

## Step 4: Find related pages
Scan wiki/index.md for related pages. Use Grep to search wiki/ for related keywords.

## Step 5: Create the note
Filename: wiki/notes/<slug>.md (slug = lowercase hyphened title)

Frontmatter:
```yaml
---
type: note
tags: [relevant, tags]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: []
---
```

Content: free-form, but should:
- State the core idea in the first paragraph
- Cross-reference related wiki pages with markdown links
- Be atomic — one idea, not a collection

## Step 6: Update related pages
For each related page identified, add a link back to this note. Even one sentence: "See also: [Note title](../notes/slug.md)."

## Step 7: Update index.md
Add the note to the appropriate section of wiki/index.md, update the count and date.

## Step 8: Commit
Run: `git add -A && git commit -m "note: <title>"`

## Step 9: Confirm
Report the note title, its path, and which pages it's now linked from/to.

</process>
