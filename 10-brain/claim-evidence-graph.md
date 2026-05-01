# Claim-Evidence Graph

## Not Just a Knowledge Graph

Traditional knowledge graphs store facts:

```
Artist:Stingray → plays_at → Venue:The_Rigger
```

The bndy brain stores claims with evidence:

```
Claim: "Stingray plays at The Rigger on 15 May 2026"
├── Evidence: Facebook paste (Signal sgnl_001)
├── Evidence: Gigantic listing (Signal sgnl_002)
├── Evidence: Venue website (Signal sgnl_003)
└── Reasoning: "Three independent sources agree"
```

## The Three-Layer Model

```
┌─────────────────────────────────────────────────────────────┐
│                    CANONICAL ENTITIES                        │
│     Venue: The Rigger │ Artist: Stingray │ Event: ...       │
└───────────────────────────────┬─────────────────────────────┘
                                │
                          derived from
                                │
┌───────────────────────────────▼─────────────────────────────┐
│                         CLAIMS                               │
│   "Stingray performs at event X"                            │
│   "Event X is hosted at The Rigger"                         │
│   "Event X is on 15 May 2026"                               │
└───────────────────────────────┬─────────────────────────────┘
                                │
                          supported by
                                │
┌───────────────────────────────▼─────────────────────────────┐
│                        EVIDENCE                              │
│   Signal 1: Facebook paste                                   │
│   Signal 2: Gigantic URL                                     │
│   Signal 3: Venue website                                    │
└─────────────────────────────────────────────────────────────┘
```

Canonical entities are the output.
Claims are the reasoning layer.
Evidence is the raw input.

## Evidence Types

| Evidence Type | Strength | Example |
|---------------|----------|---------|
| Official source | Strong | Venue's own website |
| Ticketing platform | Strong | Gigantic, Dice, Skiddle |
| Social media (verified) | Medium | Band's official Facebook |
| Social media (unverified) | Weak | Random Facebook post |
| User submission | Variable | Depends on submitter history |
| Scrape (structured) | Medium | Scraped calendar data |
| Scrape (unstructured) | Weak | Scraped text |
| Screenshot | Weak | No verification possible |
| Hearsay | Very weak | "Someone told me..." |

## Corroboration

When multiple independent sources generate the same claim:

```
Claim: "Stingray performs at The Rigger on 15 May 2026"

Evidence 1: Facebook (Band's page) - posted 1 May
Evidence 2: Gigantic (Ticketing) - listed 2 May
Evidence 3: Venue website - added 3 May

Corroboration: 3 independent sources
Independence: Different authors, different times
```

This is stronger than a single source with high confidence.

## Contradiction

When sources generate conflicting claims:

```
Claim A: "Event X is at The Rigger"
Evidence: Facebook post

Claim B: "Event X is at The Underground"
Evidence: Venue website

Status: CONTRADICTION
Resolution: Needs human review
```

Contradictions are not resolved automatically. They trigger the challenge loop.

## Claim-Evidence Relationships

### supports

Evidence directly supports a claim:

```
Signal: "STINGRAY LIVE AT THE RIGGER"
└── supports → Claim: "Stingray performs at event X"
```

### corroborates

Evidence confirms an existing claim from another source:

```
Signal 2: Gigantic listing
└── corroborates → Claim: "Stingray performs at event X"
    └── (originally from Signal 1)
```

### contradicts

Evidence conflicts with an existing claim:

```
Signal 3: Different venue mentioned
└── contradicts → Claim: "Event X at The Rigger"
```

### updates

Evidence provides new information about an existing claim:

```
Signal 4: "Doors now 7:30pm not 8pm"
└── updates → Claim: "Event X doors at 20:00"
    └── new value: 19:30
```

## Storage Model

### Evidence Table

```typescript
interface Evidence {
  evidenceId: string;        // evid_xxx
  signalId: string;          // Source signal
  claimId: string;           // Claim this supports
  relationship: 'supports' | 'corroborates' | 'contradicts' | 'updates';
  extractedText?: string;    // The specific text that generated this
  capturedAt: string;        // When evidence was captured
}
```

### DynamoDB Access Patterns

| Pattern | PK | SK |
|---------|----|----|
| Get evidence for claim | `CLAIM#claim_xxx` | `EVID#evid_xxx` |
| Get claims for signal | `SIGNAL#sgnl_xxx` | `CLAIM#claim_xxx` |
| Get contradictions | GSI: `STATUS#contradiction` | `CLAIM#claim_xxx` |

## Querying the Graph

### "Why do you think this event is at The Rigger?"

```
Query: Get evidence for claim "Event X at The Rigger"

Result:
- Evidence 1: Facebook post by Stingray, "LIVE AT THE RIGGER"
- Evidence 2: Gigantic listing, venue = "The Rigger, Newcastle-under-Lyme"
- Evidence 3: Venue website calendar entry

Reasoning: Three independent sources, all from within last 7 days
```

### "What do we know about The Rigger?"

```
Query: Get all claims where subject = "The Rigger"

Result:
- Claim: The Rigger exists (evidence: 47 signals)
- Claim: The Rigger is in Newcastle-under-Lyme (evidence: 32 signals)
- Claim: The Rigger hosts live music (evidence: 89 events)
- Claim: The Rigger books rock/indie (evidence: inferred from 67 events)
```

### "Show me uncertain claims"

```
Query: Get claims where status = 'uncertain'

Result:
- Claim: "Event X is on 15 May" - uncertainty: year not explicit
- Claim: "Artist name is 'The Wanderers'" - uncertainty: could be "Wanderers"
- Claim: "Price is £5" - uncertainty: "OTD" unclear if advance different
```

## Visual Representation

```
                         ┌──────────────────┐
                         │ Canonical Event  │
                         │ Stingray @ Rigger│
                         │ 15 May 2026      │
                         └────────┬─────────┘
                                  │
                            derived from
                                  │
              ┌───────────────────┼───────────────────┐
              │                   │                   │
              ▼                   ▼                   ▼
       ┌────────────┐     ┌────────────┐     ┌────────────┐
       │   Claim    │     │   Claim    │     │   Claim    │
       │ Artist:    │     │ Venue:     │     │ Date:      │
       │ Stingray   │     │ The Rigger │     │ 15 May     │
       └──────┬─────┘     └──────┬─────┘     └──────┬─────┘
              │                  │                  │
         supported by       supported by       supported by
              │                  │                  │
    ┌─────────┼─────────┐  ┌─────┼─────┐  ┌─────────┼────────┐
    ▼         ▼         ▼  ▼     ▼     ▼  ▼         ▼        ▼
┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
│ FB   │ │Gigan │ │Venue │ │ FB   │ │Venue │ │ FB   │ │Gigan │
│ post │ │tic   │ │ web  │ │ post │ │ web  │ │ post │ │tic   │
└──────┘ └──────┘ └──────┘ └──────┘ └──────┘ └──────┘ └──────┘
```

## Related

- [[bndy-brain-concept]] - The overall model
- [[signal-to-claim-model]] - How signals become claims
- [[memory-and-reasoning-model]] - How reasoning is stored

