# Event Candidate Model

**Last updated:** 2026-05-04
**Status:** Phase A Complete

---

## Overview

Event candidates are proposed events that emerge from signal interpretation. The LLM directly proposes event candidates during interpretation (AI-native), rather than code aggregating claims after the fact (legacy ETL thinking).

**Key principle:** LLM interprets and proposes, code validates and persists.

---

## Schema

```typescript
interface EventCandidate {
  candidateId: string;           // cand_xxxxxxxx
  candidateType: 'event';
  signalId: string;              // Source signal
  interpretationId: string;      // Source interpretation

  // LLM-proposed fields
  proposedName: string;
  proposedDate?: string;         // YYYY-MM-DD
  proposedTime?: string;         // HH:mm
  proposedVenueId?: string;      // Entity-resolved venue ID
  proposedArtistIds: string[];   // Entity-resolved artist IDs

  // LLM reasoning - REQUIRED for explainability (Brain principle)
  reasoning: string;

  // Evidence chain
  sourceClaims: ClaimReference[];

  // Completeness assessment
  completeness: 'partial' | 'complete';
  missingFields: string[];

  // Ambiguity tracking
  ambiguities: Ambiguity[];

  // Trust level
  verificationStatus: 'unverified' | 'submitter_verified';
  submitterId?: string;

  // Lifecycle
  status: 'proposed' | 'ratified' | 'rejected' | 'merged';
  createdAt: string;
  updatedAt: string;
  ratifiedAt?: string;
  ratifiedBy?: string;

  // Merge tracking
  mergedInto?: string;           // cand_xxxxxxxx if merged
}

interface ClaimReference {
  claimId: string;
  claimType: string;
  value: string;
  status: 'proposed' | 'accepted' | 'challenged';
}

interface Ambiguity {
  ambiguityType: 'entity_match' | 'date_uncertain' | 'conflicting' | 'incomplete';
  description: string;
  affectedClaimIds: string[];
  suggestedResolution?: string;
}
```

---

## AI-Native Flow

The LLM proposes event candidates during interpretation:

```
Signal submitted
    ↓
Deterministic extraction (text, tables, dates)
    ↓
LLM interpretation:
    - Generates claims (fine-grained facts)
    - Proposes eventCandidates (AI-native)
    - Identifies clarificationQuestions
    ↓
Code validates and persists:
    - Entity resolution (venue name → venueId)
    - Ambiguity detection (multiple matches)
    - DynamoDB storage
    ↓
Ratification via chat
    ↓
Canonical event created (status: draft, eventStatus: tentative)
```

**What LLM does:**
- Interprets the signal content
- Proposes event candidates with reasoning
- Explains uncertainty/ambiguity
- Identifies what needs human clarification

**What code does:**
- Validates schema compliance
- Persists proposals to DynamoDB
- Links to existing entities (entity resolution)
- Creates clarification requests
- Prevents unsafe mutation

---

## Entity Resolution

When LLM proposes `proposedVenueName: "The Rigger"`, code performs entity resolution:

```
Single match → proposedVenueId: 'vnue_abc12345'
Multiple matches → ambiguity added, venueId left empty
No match → will be created when ratified
```

Same for artists: `proposedArtistNames` → `proposedArtistIds[]`

---

## Completeness Calculation

A candidate is `complete` when all required fields are present:
- proposedName (required)
- proposedDate (required)
- proposedVenueId (required - entity linked)
- proposedArtistIds (at least one)

Missing any of these → `partial` with `missingFields` populated.

---

## Ratification Flow

```
POST /candidates/{candidateId}/ratify

Prerequisites:
- status must be 'proposed'
- completeness must be 'complete'
- ambiguities must be empty (all resolved)

On ratify:
1. Update candidate status → 'ratified'
2. Create canonical event (evnt_xxxxxxxx) with:
   - status: 'draft' (not published - needs corroboration or review)
   - eventStatus: 'tentative' (single source can't confirm)
   - verificationStatus: carried from candidate
   - evidence: from sourceClaims
3. Link venue and artist relationships
```

**Note:** Ratification creates a draft event, not a published one. Single-source events are tentative until corroborated (Phase B).

---

## Rejection Flow

```
POST /candidates/{candidateId}/reject

Body: { reason: "Duplicate event" }

On reject:
1. Update candidate status → 'rejected'
2. Store rejection reason
```

---

## DynamoDB Access Patterns

| Pattern | PK | SK | GSI |
|---------|----|----|-----|
| Get candidate | CANDIDATE#cand_xxx | #METADATA | - |
| List by signal | SIGNAL#sgnl_xxx | CANDIDATE#cand_xxx | GSI1 |
| List by status | STATUS#proposed | CANDIDATE#cand_xxx | GSI1 |

---

## Related

- [[interpretation-model]] - Where candidates are proposed
- [[../04-architecture/ingestion-pipeline]] - Overall flow
- [[../03-backlog/next-5-phases]] - Phase A definition
- [[evidence-pack-model]] - Corroboration (Phase B)
- [[clarification-model]] - Ambiguity resolution (Phase C)
