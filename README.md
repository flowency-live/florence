# bndy brain

The canonical knowledge store for bndy - an AI-native grassroots music discovery platform.

## What This Is

A markdown-native operating memory for bndy, designed to be:

- **Local-first** - No vendor lock-in, works offline
- **AI-readable** - Structured for Claude Code, Cursor, ChatGPT
- **Version-controlled** - Git-backed, every change tracked
- **Human-friendly** - Opens in Obsidian with backlinks and graph view

## How to Use

Open this folder in:

| Tool | Purpose |
|------|---------|
| **Obsidian** | Browse, link, search, graph view |
| **VSCode / Claude Code** | Build agents, write code, execute |
| **ChatGPT / Claude** | Strategy, design, reasoning, review |

## Structure

```
bndy-brain/
├── 01-strategy/     # Vision, positioning, AI-native model
├── 02-product/      # Personas, features, principles
├── 03-backlog/      # Now, next, later, risks
├── 04-architecture/ # AWS stack, pipelines, data model
├── 05-entities/     # Venue, artist, event, community-builder
├── 06-decisions/    # ADRs - architecture decision records
└── 07-prompts/      # AI tool rules and system prompts
```

## Navigation

Start with [[00-index]] for the full graph entry point.

## Principles

1. **Markdown is truth** - If it's not in markdown, it doesn't exist
2. **One repo, many tools** - Same source opened everywhere
3. **Decisions are recorded** - Every big choice becomes an ADR
4. **Don't over-engineer** - No vector DBs, RAG, MCP until needed
5. **Weekly maintenance** - Keep docs clean, archive stale content
