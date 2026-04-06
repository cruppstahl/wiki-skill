# wiki-skill

A personal wiki system for [Claude Code](https://claude.ai/code), built around the idea that the LLM owns the bookkeeping and you own the content.

You curate sources and ask questions. Claude creates pages, maintains cross-references, and keeps everything consistent.

Based on Andrej Karpathy's idea: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f

## What's included

```
.claude/commands/wiki/   ← four Claude Code slash commands
CLAUDE.md                ← wiki schema (page types, conventions, workflows)
wiki/                    ← your wiki (LLM-maintained)
  index.md               ← master catalog
  overview.md            ← high-level synthesis
  notes/                 ← atomic zettelkasten notes
  sources/               ← summaries of ingested sources
  concepts/              ← concept pages
  entities/              ← people, companies, organisations
  incubator/             ← active projects (each in its own subdirectory)
  archive/               ← concluded projects
  queries/               ← filed Q&A pages
raw/                     ← your source documents (read-only, never modified by Claude)
  clippings/             ← fetched web articles
```

Add your own areas under `raw/` and `wiki/` as needed — the system discovers them dynamically.

## Setup

### Option A — project-local skills (recommended)

Clone this repo as your wiki directory and open it in Claude Code. The `.claude/commands/wiki/` skills are picked up automatically from the project root.

```bash
git clone https://github.com/cruppstahl/wiki-skill ~/wiki
cd ~/wiki
claude  # or open in Claude Code IDE extension
```

### Option B — global skills

Copy the skill files into your global Claude config so they work in any project:

```bash
cp -r .claude/commands/wiki ~/.claude/commands/
```

Then use this repo's `CLAUDE.md` and directory scaffold as your wiki root.

## Slash commands

| Command | What it does |
|---|---|
| `/wiki:ingest <path or URL>` | Ingest a local file or web page into the wiki |
| `/wiki:query <question>` | Answer a question by synthesizing across wiki pages |
| `/wiki:new-note <title>` | Create a new atomic note with cross-references |
| `/wiki:lint` | Health-check the wiki — orphans, missing links, contradictions |

## Workflows

### Ingest a local file
```
/wiki:ingest raw/clippings/my-article.md
```

### Ingest a URL
```
/wiki:ingest https://example.com/interesting-article
```

### Ask a question
```
/wiki:query what do I know about network effects?
```

### Create a note
```
/wiki:new-note thoughts on compounding
```

### Health-check
```
/wiki:lint
```

## How it works

**CLAUDE.md** is the schema — it defines page types (source, concept, entity, note, query, project, investment), required sections, naming conventions, and the workflows Claude follows for each operation. Claude reads it at the start of every operation.

**`raw/`** is read-only source material: articles you've saved, documents you've written, data files. Claude never modifies files here.

**`wiki/`** is fully LLM-maintained. Claude creates pages, updates them when new sources add nuance or contradiction, and maintains cross-references across the graph. Every operation ends with a git commit.

**The value compounds.** Each ingested source updates all relevant wiki pages. Each query synthesizes across everything. The wiki gets more useful the more you put into it.

## Customising the schema

`CLAUDE.md` is plain Markdown — edit it to add new page types, change naming conventions, or add areas specific to your use case. Claude will follow whatever schema it finds there.

## License

MIT
