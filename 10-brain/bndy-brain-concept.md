# bndy brain Concept

## The Core Shift

The bndy brain is not an algorithm. It is an agentic knowledge system.

**Old world:**
```
rules → score → workflow
```

**AI-native:**
```
raw signal → model interprets context → proposes world-state change → evidence is retained → human/agent accepts, rejects or challenges → memory updates
```

## Not a Parser

When a source lands (poster, URL, Facebook text, spreadsheet, screenshot), the system does not ask:

> "Which parser should process this?"

It asks:

> "What does this appear to tell us about the live music world?"

## The Five-Step Loop

### 1. Evidence arrives

Anything can be dropped in:
- Poster image
- Facebook event text
- Venue website URL
- Promoter spreadsheet
- Screenshot
- Overheard conversation note
- Email forward

No special format. No required fields. Just evidence.

### 2. An AI agent reads it

The agent interprets the evidence and identifies **claims**:

```
Claim: Stingray are playing The Rigger on 15 May 2026
Claim: The Rigger is the venue
Claim: Stingray is the artist
Claim: This looks like a ticketed gig
Claim: Gigantic is a corroborating source
```

Claims are not extracted fields. They are interpretations of what the evidence appears to tell us.

### 3. The agent compares against memory

Not by hard-coded confidence maths first.

By asking:

- Do we already know these entities?
- Does this contradict anything?
- Does this strengthen an existing event?
- Does this create a new thing?
- Is this source trustworthy?
- What should change in the knowledge graph?

This is reasoning, not calculation.

### 4. It proposes a graph update

The agent proposes changes:

- Create event
- Update event
- Link artist to event
- Link event to venue
- Attach source evidence
- Flag uncertain claims
- Ask human where needed

These are proposed changes, not automatic writes.

### 5. bndy remembers the reasoning

Not just the output.

> Why did we link this to The Rigger?

Because:
- The Facebook event mentions "The Rigger, Newcastle-under-Lyme"
- The Gigantic URL confirms the same venue/date
- The venue's own history shows regular Thursday gigs
- Three sources converge on the same claim

**That reasoning is stored.** It can be queried. It can be challenged. It can be updated.

## What This Enables

### Explainable decisions

"Why do you think this is The Rigger?"

Because of evidence X, Y, Z.

### Challenge and correction

"Actually, there are two venues called The Rigger."

Memory updates. Future claims re-evaluated.

### Learning over time

Sources that are frequently challenged get lower weight.
Sources that corroborate frequently get higher trust.
Not by formula. By memory.

### Natural language queries

"What do we know about Stingray?"

The brain can answer with:
- The claims we've collected
- The evidence behind them
- The uncertainty we have
- The relationships we've inferred

## What This Is Not

| Old Model | AI-Native Model |
|-----------|-----------------|
| Parser per source type | Agent interprets any source |
| Confidence score formula | Reasoning about evidence |
| Entity resolution algorithm | Memory comparison |
| Workflow automation | Proposal and challenge loop |
| Data extraction | Claim generation |
| Field matching | World-state updates |

## The Brain vs The Database

The DynamoDB tables store facts.

The brain stores:
- Claims
- Evidence
- Reasoning
- Challenges
- Memory updates

The database is the output.
The brain is the process.

## Related

- [[signal-to-claim-model]] - How signals become claims
- [[claim-evidence-graph]] - How claims link to evidence
- [[agentic-intake-loop]] - The agent's decision process
- [[memory-and-reasoning-model]] - How bndy remembers
- [[human-challenge-loop]] - How humans correct the brain

