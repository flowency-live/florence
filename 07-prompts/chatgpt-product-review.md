# ChatGPT Product Review

Prompts for using ChatGPT (or Claude web) for product strategy and review.

## Setup

Upload these files at the start of a session:

1. `01-strategy/bndy-strategy.md`
2. `02-product/vision.md`
3. `02-product/personas.md`
4. `02-product/principles.md`
5. Current focus area docs

## Strategy Review Prompt

```
You are a product strategist helping me think through bndy, a grassroots music discovery platform.

I've uploaded our strategy and product docs. Please:

1. Identify any gaps or contradictions in our strategy
2. Challenge assumptions that might be wrong
3. Suggest areas we haven't considered
4. Rate our clarity (1-10) and explain

Be direct. I want honest feedback, not validation.
```

## Feature Prioritisation Prompt

```
Review our backlog (bndy-brain/03-backlog/).

For each item in "now.md", evaluate:
- Does this align with our stated vision?
- Is this the highest-impact work we could do?
- Are there dependencies we're missing?
- What's the risk if we delay this?

Recommend any re-prioritisation with reasoning.
```

## Persona Validation Prompt

```
Review our personas in 02-product/personas.md.

Questions:
1. Are these personas distinct enough?
2. Are we missing any important persona?
3. Do our features map well to persona needs?
4. Which persona should we focus on first and why?
```

## Competitive Analysis Prompt

```
Based on our strategy (bndy-strategy.md), analyse our competitive position.

Consider:
- Direct competitors (event discovery apps)
- Indirect competitors (Facebook Events, Instagram, Spotify)
- Potential future competitors

For each:
- What's their strength?
- What's their weakness?
- Where do we have an advantage?
- Where are we vulnerable?
```

## ADR Review Prompt

```
I'm considering this architecture decision:

[Paste decision context]

Options:
1. [Option A]
2. [Option B]
3. [Option C]

Help me think through:
- What am I not considering?
- What are the long-term implications?
- Which option best fits our AI-native strategy?
- What would you recommend and why?
```

## Weekly Review Prompt

```
It's time for our weekly product review.

This week:
- [What we shipped]
- [What we learned]
- [What blocked us]

Questions:
1. Are we still on strategy?
2. Should we adjust priorities?
3. What should we focus on next week?
4. Any risks emerging?
```

## User Research Synthesis Prompt

```
I conducted user research this week. Here are the raw notes:

[Paste notes]

Please:
1. Identify key themes
2. Map findings to our personas
3. Highlight surprises or contradictions
4. Recommend actions
```

## Tips for Effective Sessions

1. **Upload docs fresh** - Don't assume context from previous chats
2. **Be specific** - "Review our venue model" not "help with product"
3. **Ask for pushback** - Explicitly request challenge
4. **One topic per session** - Context windows get confused
5. **Export insights** - Save useful outputs back to bndy-brain

## Related

- [[claude-code-rules]]
- [[cursor-rules]]
