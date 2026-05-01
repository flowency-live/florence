# ADR-001: Obsidian + Git as Local AI Brain

## Status

**Status:** Accepted
**Date:** 01/05/2026
**Deciders:** Jason

## Context

bndy needs a knowledge management system that:
- Stores product strategy, architecture, and decisions
- Is readable by AI tools (Claude Code, Cursor, ChatGPT)
- Supports version control and collaboration
- Has no vendor lock-in
- Works offline
- Scales from solo founder to team

Options considered:
1. Notion
2. Confluence
3. Obsidian + Git
4. Custom database + RAG
5. Plain markdown in repo

## Decision

We decided to use Obsidian with Git-backed markdown because:

> In the context of bndy's AI-native operating model,
> facing the need for a durable, AI-readable knowledge store,
> we decided to use Obsidian + Git with structured markdown,
> to achieve local-first, versionable, tool-agnostic documentation,
> accepting that discipline is required to maintain doc quality.

## Consequences

### Positive

- **AI-readable**: Markdown files work directly with Claude Code, Cursor, ChatGPT
- **Local-first**: Works offline, no dependency on external services
- **Version-controlled**: Full history, branching, collaboration via Git
- **No vendor lock-in**: Plain text, portable anywhere
- **Fast**: Obsidian is extremely fast for search, navigation, linking
- **Graph view**: Visualise relationships between concepts
- **Low cost**: Free (Obsidian) + free (Git hosting)

### Negative

- **Manual discipline**: No automated structure enforcement
- **Learning curve**: Team needs to learn Obsidian and Git
- **No real-time collaboration**: Git merge conflicts if editing same file
- **No permissions**: Everyone with repo access sees everything
- **Maintenance burden**: Docs can become stale without attention

### Neutral

- Need to establish conventions for file naming, linking, structure
- Weekly "brain maintenance" ritual recommended

## Alternatives Considered

### Option 1: Notion

**Pros:**
- Real-time collaboration
- Nice UI, familiar to many
- Some AI features built-in

**Cons:**
- Vendor lock-in (export is lossy)
- AI tools can't easily read Notion pages
- Paid for team features
- Slower for large workspaces

**Why rejected:** AI tools need direct file access, not API wrappers.

### Option 2: Confluence

**Pros:**
- Enterprise standard
- Good permissions model

**Cons:**
- Heavy, slow
- Expensive
- Poor markdown support
- Difficult for AI tools to access

**Why rejected:** Over-engineered for current needs, poor AI compatibility.

### Option 3: Custom RAG system

**Pros:**
- Optimised for AI retrieval
- Could be very powerful

**Cons:**
- Significant build effort
- Maintenance overhead
- Premature optimisation

**Why rejected:** Build product first, add RAG if needed later.

### Option 4: Plain markdown in code repo

**Pros:**
- Simple
- Co-located with code

**Cons:**
- No graph/backlinks
- Mixed concerns (docs + code)
- Less pleasant to browse

**Why rejected:** Obsidian adds significant value for minimal cost.

## Related

- [[01-strategy/ai-native-reframe|AI-Native Reframe]]
- [[README]]
