# Human Challenge Loop

## The Brain Can Be Wrong

The bndy brain makes interpretations, not ground truth. It can be:

- **Mistaken** - misread a date, confused two venues
- **Incomplete** - missing context a human would know
- **Outdated** - world changed since evidence was captured
- **Biased** - pattern-matched on insufficient data

Humans are the correction mechanism.

## Challenge, Not Override

Traditional systems have "edit" buttons. You change the data.

The bndy brain has challenges. You question the reasoning.

| Traditional | bndy brain |
|-------------|------------|
| Edit venue name | "I think this is a different venue" |
| Change event date | "The date is wrong because..." |
| Delete duplicate | "These are actually the same event" |
| Mark as spam | "This isn't a real event" |

Challenges capture **why** the brain was wrong, not just **what** to fix.

## Challenge Types

### Factual challenge

> "The date is wrong. It's 22 May, not 15 May."

The brain responds:
- Updates the claim
- Stores the correction
- Notes: "Date OCR may be unreliable"
- Adjusts confidence in similar future extractions

### Entity challenge

> "This isn't The Rigger in Newcastle. It's The Rigger in London."

The brain responds:
- Creates (or links to) the correct venue
- Stores: "Multiple venues share this name"
- Updates matching rules to require location disambiguation
- Re-checks other events that matched to Newcastle Rigger

### Existence challenge

> "This event isn't happening. It was cancelled."

The brain responds:
- Marks event as cancelled (not deleted)
- Stores: "Source didn't indicate cancellation"
- Notes: Facebook posts don't always reflect cancellations
- Future: Check multiple sources before confirming events

### Relationship challenge

> "Stingray isn't the headliner. They're the support act."

The brain responds:
- Updates the event structure
- Stores: "Poster layout was misleading"
- Notes: Top-of-poster ≠ always headliner
- Adjusts future interpretation

### Source challenge

> "This Facebook page isn't the real band. It's a fan page."

The brain responds:
- Downgrades source reliability
- Re-evaluates claims from this source
- Stores: "Verified ≠ official for Facebook pages"
- Updates verification checks

## The Challenge Interface

```
┌─────────────────────────────────────────────────────────────────────┐
│ Challenge Claim                                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│ Claim: "Event: Stingray @ The Rigger, 15 May 2026"                  │
│                                                                     │
│ Brain's reasoning:                                                  │
│ "I matched this to The Rigger in Newcastle-under-Lyme because       │
│  the Facebook post said 'The Rigger' and the postcode ST5 was       │
│  mentioned in a comment."                                           │
│                                                                     │
│ ┌─────────────────────────────────────────────────────────────────┐ │
│ │ What's wrong?                                                   │ │
│ │                                                                 │ │
│ │ ○ Wrong venue (different location)                              │ │
│ │ ○ Wrong date                                                    │ │
│ │ ○ Wrong artist                                                  │ │
│ │ ○ Event doesn't exist                                           │ │
│ │ ○ These are duplicates                                          │ │
│ │ ○ Other                                                         │ │
│ └─────────────────────────────────────────────────────────────────┘ │
│                                                                     │
│ ┌─────────────────────────────────────────────────────────────────┐ │
│ │ Explain what's correct:                                         │ │
│ │                                                                 │ │
│ │ This is actually The Rigger in London, not Newcastle.          │ │
│ │ The band is from London and rarely travels north.              │ │
│ │ ___________________________________________________________     │ │
│ └─────────────────────────────────────────────────────────────────┘ │
│                                                                     │
│                                          [Submit Challenge]         │
└─────────────────────────────────────────────────────────────────────┘
```

## Challenge Resolution

After a challenge is submitted:

### 1. Record the challenge

```typescript
interface Challenge {
  challengeId: string;
  claimId: string;
  challengedBy: string;
  challengedAt: string;

  challengeType: ChallengeType;
  explanation: string;

  // What the human says is correct
  correction?: {
    correctValue: unknown;
    supportingEvidence?: string;
  };

  status: 'pending' | 'accepted' | 'rejected' | 'escalated';
}
```

### 2. Evaluate the challenge

The brain asks:
- Does this challenge have supporting evidence?
- Does it contradict multiple strong sources?
- Is the challenger a trusted user?
- Has this pattern been challenged before?

### 3. Accept or escalate

**Auto-accept when:**
- Challenger provides clear evidence
- Only one source supported the original claim
- Challenge aligns with known uncertainty

**Escalate when:**
- Multiple strong sources conflict with challenge
- Challenge would invalidate many dependent claims
- Pattern is novel (first time seeing this issue)

### 4. Update memory

On acceptance:
- Mark original claim as invalidated
- Create new claim with correction
- Store the challenge as a learning
- Update source reliability
- Re-evaluate dependent claims

## Cascading Updates

When a challenge invalidates a claim:

```
Challenge: "This isn't The Rigger in Newcastle"
           │
           ├── Invalidate: Event X venue = The Rigger Newcastle
           │
           ├── Check: Other events at "The Rigger" from this source
           │   └── 2 events found - flag for review
           │
           ├── Update: Source reliability for original signal
           │   └── Decrease by 0.1 (location error)
           │
           └── Learn: "The Rigger" requires location disambiguation
               └── Future matches must confirm location
```

## Challenge Patterns → Learnings

Over time, challenges reveal patterns:

### Common mistakes

```
Learning: Date OCR has 15% error rate on hand-drawn posters
Action: Lower confidence on dates from poster images
```

### Source reliability

```
Learning: Facebook fan pages are often mistaken for official
Action: Require corroboration before trusting unverified pages
```

### Entity ambiguity

```
Learning: 3 venues called "The Crown" in Greater Manchester
Action: Always require postcode or town for Crown matches
```

### Seasonal patterns

```
Learning: December events often rescheduled
Action: Flag December events for freshness checks
```

## Challenger Trust

Not all challengers are equal:

| Challenger Type | Trust Level | Action |
|-----------------|-------------|--------|
| Venue owner | High | Auto-accept on their venue |
| Artist | High | Auto-accept on their events |
| Regular correct user | Medium | Weighted consideration |
| New user | Low | Require evidence |
| Known bad actor | None | Ignore or escalate |

Trust is earned by challenge accuracy over time.

## The Loop Completes

```
Signal arrives
     │
     ▼
Brain interprets → generates claims
     │
     ▼
Brain compares with memory
     │
     ▼
Brain proposes world-state change
     │
     ├── High confidence → Auto-accept
     │
     └── Low confidence → Human review
                              │
                              ▼
                         Human accepts
                              │
                              └── OR challenges
                                       │
                                       ▼
                              Challenge recorded
                                       │
                                       ▼
                              Brain learns
                                       │
                                       ▼
                              Memory updates
                                       │
                                       ▼
                         Future signals benefit
                                       │
                                       └─────────────────┐
                                                         │
Signal arrives ←─────────────────────────────────────────┘
```

The brain gets smarter with every challenge.

## Anti-Patterns

### Don't just override

> "I changed the venue to The Rigger London"

This fixes the data but doesn't teach the brain.

**Better:**
> "The brain matched to Newcastle because of ST5 postcode in comments, but that was a different event. London artists typically play London venues."

### Don't delete, invalidate

> "Delete this duplicate event"

This loses history.

**Better:**
> "These are the same event. Keep the one with more evidence, mark the other as duplicate, record why."

### Don't ignore uncertainty

> "Whatever, approve it"

This pollutes the data.

**Better:**
> "I'm not sure either. Flag for later review when more evidence arrives."

## Related

- [[bndy-brain-concept]] - The overall model
- [[agentic-intake-loop]] - The agent's decision process
- [[memory-and-reasoning-model]] - How the brain remembers

