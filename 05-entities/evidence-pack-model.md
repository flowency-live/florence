# Evidence Pack Model

## Purpose

An EvidencePack groups multiple Signals and their Interpretations that corroborate the same knowledge.

**Evidence Packs are the central cognitive structure.**

The real flow is:

```
Signals
→ Interpretations
→ Claims
→ Evidence Pack (THE COGNITIVE CORE)
→ Candidate Entities
→ Ratification
→ Canonical Knowledge
```

High-confidence knowledge emerges from multiple independent sources agreeing.

## Core Principle

```
Single Signal = weak evidence
Multiple Signals agreeing = strong evidence
```

An event might be supported by:
- Facebook event text
- Poster image
- Venue website calendar
- Gigantic ticket link
- Community builder confirmation

That combination creates high-confidence knowledge.

## TypeScript Interface

```typescript
interface EvidencePack {
  packId: string;                 // pack_xxxxxxxx

  // What this pack supports
  proposition: string;            // "Stingray plays The Rigger on 2026-05-15"
  propositionType: PropositionType;

  // What this pack is about (legacy, being replaced by proposition)
  subject: PackSubject;

  // Contributing evidence
  signals: SignalReference[];
  interpretations: InterpretationReference[];
  claims: ClaimReference[];

  // Corroboration assessment
  corroborationStrength: CorroborationStrength;
  corroborationReasoning: string;

  // Ambiguities requiring resolution
  ambiguities: Ambiguity[];
  clarificationIds: string[];     // Open clarification requests

  // Output references
  candidateEntityIds: string[];   // Candidate entities this pack supports
  proposedRelationshipIds: string[]; // Proposed relationships

  // What this evidence proposes (being replaced by candidateEntityIds)
  proposedEntities: ProposedEntities;

  // Lifecycle
  status: PackStatus;
  createdAt: string;
  updatedAt: string;
  ratifiedAt?: string;            // When human confirmed
  publishedAt?: string;
}

type PropositionType =
  | 'event'           // An event is happening
  | 'artist_venue'    // Artist plays at venue
  | 'venue_location'  // Venue is at location
  | 'artist_identity' // Artist is this entity
  | 'venue_identity'; // Venue is this entity

interface Ambiguity {
  ambiguityType: AmbiguityType;
  description: string;            // "Multiple venues named The Rigger"
  affectedClaimIds: string[];
  suggestedResolution?: string;
}

type AmbiguityType =
  | 'entity_match'    // Multiple entities could match
  | 'date_uncertain'  // Year or date unclear
  | 'conflicting'     // Evidence disagrees
  | 'incomplete';     // Missing required data

interface PackSubject {
  type: 'event' | 'venue' | 'artist';
  description: string;            // "Stingray @ The Rigger, 15 May 2026"
  candidateEntityId?: string;     // If matches existing entity
}

interface SignalReference {
  signalId: string;
  signalType: string;
  addedAt: string;
  contribution: string;           // What this signal adds
}

interface InterpretationReference {
  interpretationId: string;
  signalId: string;
  version: number;
  claimCount: number;
}

interface ClaimReference {
  claimId: string;
  claimType: string;
  value: string;
  status: 'proposed' | 'accepted' | 'challenged';
}

// NOT numeric - categorical strength
type CorroborationStrength = 'weak' | 'moderate' | 'strong';

interface ProposedEntities {
  events: ProposedEvent[];
  venues: ProposedVenue[];
  artists: ProposedArtist[];
  relationships: ProposedRelationship[];
}

interface ProposedEvent {
  name: string;
  date: string;
  venueRef: string;               // Existing venue ID or proposed
  artistRefs: string[];           // Existing artist IDs or proposed
  confidence: CorroborationStrength;
}

interface ProposedVenue {
  name: string;
  location?: string;
  matchedEntityId?: string;       // If matches existing
  isNew: boolean;
}

interface ProposedArtist {
  name: string;
  matchedEntityId?: string;
  isNew: boolean;
}

interface ProposedRelationship {
  type: string;                   // "artist_performs_at", "venue_hosts"
  fromEntity: string;
  toEntity: string;
}

type PackStatus =
  | 'gathering'                   // Still collecting evidence
  | 'ready_for_review'            // Has enough evidence
  | 'under_review'                // Human reviewing
  | 'published'                   // Entities created
  | 'rejected';                   // Insufficient evidence
```

## Corroboration Strength

**NOT a numeric score.** Categorical assessment with reasoning.

| Strength | Definition | Example |
|----------|------------|---------|
| `weak` | Single source, no corroboration | Just a Facebook post |
| `moderate` | 2 sources OR 1 highly trusted source | Facebook + poster image |
| `strong` | 3+ independent sources agree | Facebook + Gigantic + venue website |

**Always includes reasoning:**

```typescript
corroborationStrength: 'strong',
corroborationReasoning: 'Three independent sources confirm: Facebook event (band page), Gigantic ticket listing, and The Rigger website calendar all show same event, date, and artists.'
```

## ID Format

```
pack_[8-char-nanoid]

Examples:
pack_a1b2c3d4
pack_x9y8z7w6
```

## Example: Building an EvidencePack

### Signal 1: Facebook paste

```
User pastes: "STINGRAY LIVE AT THE RIGGER THURSDAY 15TH MAY 8PM"

Signal: sgnl_001
Interpretation: intp_001
Claims:
  - event_exists: "Stingray Live"
  - artist_performs: "Stingray"
  - venue_hosts: "The Rigger" (uncertain)
  - event_date: "2026-05-15" (inferred)
```

**EvidencePack created:**
```typescript
{
  packId: "pack_abc123",
  subject: {
    type: "event",
    description: "Stingray @ The Rigger, 15 May 2026"
  },
  signals: [{ signalId: "sgnl_001", contribution: "Primary announcement" }],
  corroborationStrength: "weak",
  corroborationReasoning: "Single source (Facebook paste). Venue match uncertain.",
  status: "gathering"
}
```

### Signal 2: Poster image

```
User uploads poster with same event details

Signal: sgnl_002
Interpretation: intp_002
Claims confirm same event
```

**EvidencePack updated:**
```typescript
{
  signals: [
    { signalId: "sgnl_001", contribution: "Primary announcement" },
    { signalId: "sgnl_002", contribution: "Visual confirmation" }
  ],
  corroborationStrength: "moderate",
  corroborationReasoning: "Two sources agree: Facebook text and poster image both show Stingray at The Rigger on 15 May.",
  status: "gathering"
}
```

### Signal 3: Gigantic URL

```
User pastes Gigantic ticket link

Signal: sgnl_003
Interpretation: intp_003
Claims confirm + add ticket source
```

**EvidencePack now strong:**
```typescript
{
  signals: [
    { signalId: "sgnl_001", contribution: "Primary announcement" },
    { signalId: "sgnl_002", contribution: "Visual confirmation" },
    { signalId: "sgnl_003", contribution: "Ticket source confirmation" }
  ],
  corroborationStrength: "strong",
  corroborationReasoning: "Three independent sources: Facebook, poster, and Gigantic ticket link all confirm same event details.",
  proposedEntities: {
    events: [{
      name: "Stingray Live",
      date: "2026-05-15",
      venueRef: "vnue_rigger_123",
      artistRefs: ["arts_stingray_456"],
      confidence: "strong"
    }]
  },
  status: "ready_for_review"
}
```

## Pack Lifecycle

```
Signal arrives → First interpretation
↓
EvidencePack created
Status: GATHERING
Strength: WEAK
↓
More signals added
Strength: MODERATE
↓
Sufficient corroboration
Status: READY_FOR_REVIEW
Strength: STRONG
↓
Human reviews
├── Approve → Status: PUBLISHED
│   └── Canonical entities created
└── Reject → Status: REJECTED
    └── Pack archived
```

## DynamoDB Access Patterns

| Pattern | PK | SK |
|---------|----|----|
| Get pack | `PACK#pack_xxx` | `#METADATA` |
| List by status | GSI: `STATUS#ready_for_review` | `PACK#pack_xxx` |
| List by strength | GSI: `STRENGTH#strong` | `PACK#pack_xxx` |
| Get signals in pack | `PACK#pack_xxx` | `SIGNAL#sgnl_xxx` |

## Automatic Pack Creation

When a new interpretation is created:

1. Check if claims match existing pack subject
2. If yes → Add signal to existing pack
3. If no → Create new pack with this signal

**Matching logic:**
- Same venue name + similar date range
- Same artist + same venue + same date
- High claim overlap

## Manual Pack Linking

Human can also:
- Merge two packs (they're about same event)
- Split a pack (actually two different events)
- Add signal to pack manually

## Phase 1 Scope

In Phase 1:
- Packs are created but **not auto-published**
- All packs go to human review
- No auto-merge of packs

Later phases add automation.

## Related

- [[signal-model]] - Raw evidence
- [[interpretation-model]] - Understanding of evidence
- [[claim-model]] - Proposed world-state changes
- [[clarification-model]] - Ambiguity resolution
- [[relationship-model]] - Proposed relationships
- [[../03-backlog/next-5-phases]] - Phase B: Evidence Packs
- [[../10-brain/claim-evidence-graph|Claim-Evidence Graph]] - How claims link to evidence
- [[../11-runtime/cognitive-runtime|Cognitive Runtime]] - How packs evolve

