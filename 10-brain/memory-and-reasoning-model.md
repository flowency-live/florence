# Memory and Reasoning Model

## The Brain Remembers Reasoning

Traditional databases store facts:

```
event_id: evnt_123
venue_id: vnue_456
artist_id: arts_789
date: 2026-05-15
```

The bndy brain stores facts **and** the reasoning that produced them:

```
event_id: evnt_123
venue_id: vnue_456
artist_id: arts_789
date: 2026-05-15

reasoning: {
  venue_match: {
    question: "Is this The Rigger in Newcastle-under-Lyme?",
    answer: "Yes, because the Facebook post says 'The Rigger' and the
             venue's postcode ST5 matches Newcastle-under-Lyme",
    evidence: ["sgnl_001", "sgnl_002"],
    confidence: "high"
  },
  artist_match: {
    question: "Is this the same Stingray we've seen before?",
    answer: "Yes, same Facebook page, same genre, same typical venues",
    evidence: ["sgnl_001", "prior_events"],
    confidence: "high"
  },
  date_inference: {
    question: "What year is 'Thursday 15th May'?",
    answer: "2026, because that's when Thursday falls on 15th May
             and the event hasn't happened yet",
    evidence: ["calendar_check"],
    confidence: "high"
  }
}
```

## Why Store Reasoning?

### 1. Explainability

> "Why do you think this event is at The Rigger?"

The brain can answer:

> "Because the Facebook post mentioned 'The Rigger', the Gigantic listing
> shows the same venue name, and the venue's own website has this event
> on their calendar. Three independent sources all point to the same venue."

### 2. Challenge and correction

> "Actually, there are two venues called The Rigger."

The brain can re-evaluate:

> "I matched to The Rigger in Newcastle-under-Lyme because of the ST5
> postcode. If there's another The Rigger, I need more evidence to
> distinguish them. Adding this to uncertainty flags."

### 3. Learning from mistakes

If a human corrects a decision, the brain learns:

> "I matched 'The Rig' to 'The Rigger' based on name similarity.
> Human said it's actually 'The Rig Bar' - a different venue.
> Updating: short forms are less reliable for venue matching."

### 4. Cascading updates

If a core piece of reasoning is invalidated, dependent decisions can be re-evaluated:

> "I thought Stingray always plays rock venues. New evidence shows
> they also play acoustic sets at folk venues. Re-evaluating past
> venue matches that assumed rock genre..."

## Memory Structure

### Claim Memory

Every claim is stored with its full context:

```typescript
interface ClaimMemory {
  claimId: string;

  // The claim itself
  claim: {
    type: ClaimType;
    subject: string;
    predicate: string;
    object: string;
  };

  // How we arrived at this claim
  reasoning: {
    interpretation: string;      // What the agent understood
    comparison: string;          // How it compared to memory
    decision: string;            // Why this conclusion
  };

  // Supporting evidence
  evidence: {
    signalIds: string[];
    extractedFragments: string[];
    corroboratingClaims: string[];
  };

  // Status and lifecycle
  status: 'proposed' | 'accepted' | 'challenged' | 'invalidated';
  acceptedAt?: string;
  challengedAt?: string;
  challengeReason?: string;
  resolution?: string;
}
```

### Reasoning Chains

When one claim depends on another:

```
Claim A: "The Rigger is in Newcastle-under-Lyme"
└── Reasoning: Postcode ST5, Google Maps confirms

    Claim B: "This event is at The Rigger in Newcastle-under-Lyme"
    └── Reasoning: Depends on Claim A
    └── If Claim A invalidated, re-evaluate Claim B

        Claim C: "Stingray plays venues in Staffordshire"
        └── Reasoning: Depends on Claim B (and 5 other events)
        └── If Claim B invalidated, recalculate
```

### Source Memory

Track reliability over time:

```typescript
interface SourceMemory {
  sourceType: string;           // e.g., "facebook_verified_page"
  sourceName: string;           // e.g., "Stingray Official"

  history: {
    totalClaims: number;
    acceptedClaims: number;
    challengedClaims: number;
    invalidatedClaims: number;
  };

  reliability: {
    current: number;            // 0.0 - 1.0
    trend: 'improving' | 'stable' | 'declining';
    lastUpdated: string;
  };

  learnings: string[];          // What we've learned about this source
}
```

## Querying Memory

### Natural language queries

> "What do we know about The Rigger?"

Memory retrieval:

```
Entity: The Rigger (vnue_123)
Type: Venue
Location: Newcastle-under-Lyme, ST5

Claims:
- Exists: HIGH confidence (47 events as evidence)
- Location is Newcastle-under-Lyme: HIGH (postcode, Google Place ID)
- Books rock/indie music: MEDIUM (inferred from 67% of events)
- Has ~100 capacity: LOW (one source mention)
- Open since 2018: MEDIUM (earliest event in database)

Recent events:
- 12 events in past 90 days
- Artists include: Stingray, The Wanderers, ...

Relationships:
- Frequently books: [list of artists]
- Similar to: The Underground, The Sugarmill
```

### "Why" queries

> "Why do you think Stingray is a rock band?"

Memory retrieval:

```
Claim: "Stingray plays rock music"

Reasoning:
- 3 events at rock venues (The Rigger, The Underground)
- Facebook page mentions "rock" in bio
- Spotify genre tags include "rock", "alternative"
- No contradicting evidence (no jazz venues, no classical tags)

Evidence:
- Signal sgnl_001: Facebook bio scrape
- Signal sgnl_045: Spotify API lookup
- Inferred from venue history

Confidence: MEDIUM
Uncertainty: Could also play punk, indie, or crossover genres
```

### "What if" queries

> "What if The Rigger closed?"

Memory simulation:

```
Impact analysis:

Direct:
- 47 historical events would lose venue verification
- 12 upcoming events would be flagged as uncertain

Derived:
- Artist:venue relationships involving The Rigger would be historical-only
- "Staffordshire rock scene" inferences would lose one data point
- Similar venue calculations would exclude The Rigger

Recommendation:
- If confirmed, mark venue as "closed" with date
- Historical data remains valid
- Future events at "The Rigger" should trigger review
```

## Memory Updates

### On new evidence

```
1. New signal arrives
2. Agent generates claims
3. Memory check: Do these claims exist?
4. If new: Add to memory with full reasoning
5. If existing: Update confidence, add corroboration
6. If contradicting: Flag for resolution
```

### On human correction

```
1. Human challenges claim
2. Store challenge reason
3. Re-evaluate dependent claims
4. Update source reliability
5. Store correction as learning
```

### On time passage

```
1. Events become historical (date passes)
2. Stale claims lose confidence
3. "Upcoming" becomes "past"
4. Source freshness decays
```

## Storage Implementation

### DynamoDB Patterns

| Access Pattern | PK | SK |
|----------------|----|----|
| Get claim reasoning | `CLAIM#claim_xxx` | `#REASONING` |
| Get claim evidence | `CLAIM#claim_xxx` | `EVID#evid_xxx` |
| Get entity claims | `ENTITY#vnue_xxx` | `CLAIM#claim_xxx` |
| Get source history | `SOURCE#facebook_verified` | `HIST#2026-05` |
| Get challenges | GSI: `STATUS#challenged` | `CLAIM#claim_xxx` |

### S3 for Reasoning Logs

For full reasoning chains that are too large for DynamoDB:

```
s3://bndy-brain/
├── reasoning/
│   ├── 2026/05/01/
│   │   ├── claim_xxx_reasoning.json
│   │   └── claim_yyy_reasoning.json
│   └── ...
└── corrections/
    ├── 2026/05/01/
    │   ├── challenge_001.json
    │   └── resolution_001.json
    └── ...
```

## The Difference

Traditional system:
> "We extracted these fields and ran a matching algorithm."

bndy brain:
> "We interpreted this evidence to mean X because of Y. We checked
> against what we already know. We found Z was similar. Here's why
> we concluded this, and here's what we're uncertain about."

The brain doesn't just store answers. It stores understanding.

## Related

- [[bndy-brain-concept]] - The overall model
- [[claim-evidence-graph]] - How claims link to evidence
- [[human-challenge-loop]] - How humans correct the brain

