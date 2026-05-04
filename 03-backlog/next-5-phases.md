# Next 5 Phases

**Last updated:** 2026-05-04

---

## Critical Insight

**The Brain must learn to reason before it learns to consume at scale.**

Automated ingestion multiplies:
- Ambiguity
- Duplicates
- Low-trust claims
- Conflicting evidence

Source Profiles and Automation are **LAST**, after the cognitive core is solid.

---

## Current State (Foundation Complete)

| Build | Name | Status |
|-------|------|--------|
| BUILD-001 | Signal Inbox | ✅ Deployed |
| BUILD-002 | Claim Generator | ✅ Deployed |
| BUILD-003 | Review Console | ✅ Deployed |
| BUILD-004 | Canonical Entity Drafts | ✅ Deployed |
| Chat UI | Conversational submission | ✅ Deployed |

---

## Phase Order

| Phase | Name | Why This Order |
|-------|------|----------------|
| A | Event Aggregation | Complete the candidate → entity cycle |
| B | Evidence Packs & Corroboration | The central cognitive structure |
| C | Ambiguity Resolution | First-class ambiguity model, chat as primary interface |
| D | Proposed Relationships | Graph emerges from evidence, not hard-written |
| E | Source Profiles + Automation | Scale ingestion ONLY after reasoning is solid |

---

## Terminology

| Legacy | Brain-Native |
|--------|--------------|
| Draft entity | Candidate entity, Proposed entity |
| Create entity | Entity emerges from evidence |
| Auto-create relationship | Proposed relationship strengthens with corroboration |
| Review UI | Conversational ratification |

---

## Phase A: Event Aggregation

**Goal:** Candidate events emerge from grouped claims.

### A1: Event Candidate Schema

```typescript
interface EventCandidate {
  candidateId: string;           // cand_xxxxxxxx
  candidateType: 'event';

  // Aggregated from claims
  proposedName: string;
  proposedDate?: string;
  proposedTime?: string;
  proposedVenueId?: string;      // Linked entity ID
  proposedArtistIds: string[];   // Linked entity IDs

  // Evidence
  sourceClaims: string[];
  sourceSignals: string[];

  // Confidence
  completeness: 'partial' | 'complete';
  ambiguities: Ambiguity[];

  status: 'proposed' | 'ratified' | 'rejected' | 'merged';
  createdAt: string;
}
```

### A2: Claim Aggregator

Groups related claims into event candidates:

```
Claims from Signal sgnl_123:
├── event_exists: "Stingray Live"
├── event_date: "2026-05-15"
├── venue_hosts: "The Rigger" → vnue_abc
└── artist_performs: "Stingray" → arts_xyz

↓

EventCandidate:
├── proposedName: "Stingray Live"
├── proposedDate: "2026-05-15"
├── proposedVenueId: vnue_abc
├── proposedArtistIds: [arts_xyz]
├── completeness: 'complete'
└── ambiguities: []
```

### A3: Entity Publishing

Ratified candidates become canonical entities.

```
candidate → ratified → CanonicalEvent (status: 'published')
```

### Acceptance Criteria

- [ ] Related claims grouped by signal/interpretation
- [ ] Event candidates proposed when enough data
- [ ] Venue and artist IDs linked (not names)
- [ ] Candidates can be ratified via chat
- [ ] Ratified candidates become published events

---

## Phase B: Evidence Packs & Corroboration

**Goal:** Evidence Packs become the central cognitive structure.

### The Real Flow

```
Signals
→ Interpretations
→ Claims
→ Evidence Pack (THE COGNITIVE CORE)
→ Candidate Entities
→ Ratification
→ Canonical Knowledge
```

Evidence Packs group claims that support the same world-state proposition.

### B1: Evidence Pack Schema

```typescript
interface EvidencePack {
  packId: string;                // pack_xxxxxxxx

  // Contributing evidence
  signals: string[];
  interpretations: string[];
  claims: string[];

  // What this pack supports
  proposition: string;           // "Stingray plays The Rigger on 2026-05-15"
  propositionType: 'event' | 'artist_venue' | 'venue_location';

  // Corroboration
  corroborationStrength: 'weak' | 'moderate' | 'strong';
  corroborationReasoning: string;

  // Outputs
  candidateEntityIds: string[];
  proposedRelationshipIds: string[];

  // Lifecycle
  status: 'gathering' | 'ready' | 'ratified' | 'rejected';
  createdAt: string;
  updatedAt: string;
}
```

**Strength definitions:**
- `weak`: Single source, no corroboration
- `moderate`: 2 sources OR 1 trusted source
- `strong`: 3+ independent sources agree

### B2: Pack Builder

When claims arrive, check if they corroborate existing packs or start new ones.

### B3: Verification Promotion

When pack strength increases, promote entity verification:

```
unverified → submitter_verified → community_verified → source_correlated → owner_confirmed
```

### Acceptance Criteria

- [ ] Evidence packs group corroborating claims
- [ ] Pack strength calculated correctly
- [ ] Verification status promotes with corroboration
- [ ] Multiple signals strengthen confidence

---

## Phase C: Ambiguity Resolution

**Goal:** Ambiguity becomes first-class. Chat becomes primary resolution interface.

### C1: Clarification Request Schema

```typescript
interface ClarificationRequest {
  clarificationId: string;       // clar_xxxxxxxx

  // What needs clarifying
  entityCandidateId?: string;
  claimId?: string;
  evidencePackId?: string;

  // The question
  question: string;              // "Is this The Rigger in Newcastle-under-Lyme?"
  questionType: 'entity_match' | 'date_confirm' | 'venue_location' | 'artist_identity';

  // Options if applicable
  options?: ClarificationOption[];

  // Resolution
  status: 'open' | 'resolved' | 'dismissed';
  resolvedBy?: string;
  resolution?: string;
  resolvedAt?: string;

  createdAt: string;
}

interface ClarificationOption {
  optionId: string;
  label: string;
  entityId?: string;
  confidence?: number;
}
```

### C2: Chat as Primary Interface

The magic interaction:

```
bndy:
I found two venues called "The Rigger".
Is this the one in Newcastle-under-Lyme?

user:
Yes.

bndy:
Great. I linked this event and strengthened confidence in that venue match.
```

Chat is not review. Chat is ratification.

### C3: Ambiguity Surfacing

When evidence pack has ambiguities, generate clarification requests:

```
EvidencePack:
├── proposition: "Stingray plays The Rigger"
├── ambiguities: ["Multiple venues named The Rigger"]
└── status: 'gathering'

↓

ClarificationRequest:
├── question: "Which 'The Rigger' is this?"
├── options: [vnue_123 (Newcastle), vnue_456 (Sheffield)]
└── status: 'open'
```

### Acceptance Criteria

- [ ] Ambiguities generate clarification requests
- [ ] Chat surfaces clarifications naturally
- [ ] User resolution updates entities and packs
- [ ] Resolved ambiguities strengthen confidence

---

## Phase D: Proposed Relationships

**Goal:** Relationships emerge from evidence and strengthen with corroboration.

### D1: Proposed Relationship Schema

```typescript
interface ProposedRelationship {
  relationshipId: string;        // rel_xxxxxxxx

  fromEntityId: string;
  toEntityId: string;
  relationshipType: 'performed_at' | 'hosted' | 'similar_to';

  // Evidence
  evidence: EvidenceLink[];
  evidencePackIds: string[];

  // Confidence
  strength: 'weak' | 'moderate' | 'strong';
  occurrenceCount: number;
  firstSeen: string;
  lastSeen: string;

  // Lifecycle
  status: 'proposed' | 'confirmed' | 'rejected';
  createdAt: string;
  updatedAt: string;
}
```

### D2: Relationship Strengthening

Relationships don't just "exist on publish". They:
1. Start as `proposed` (weak) from first evidence
2. Strengthen with corroboration
3. Promote to `confirmed` when strong

```
First signal: Stingray → The Rigger (weak)
Second signal: confirms same (moderate)
Third source: confirms again (strong → confirmed)
```

### D3: Relationship Queries

Only return relationships above threshold:

```
GET /artists/{id}/venues?minStrength=moderate
```

### Acceptance Criteria

- [ ] Relationships start as proposed
- [ ] Corroboration strengthens relationships
- [ ] Query API respects strength thresholds
- [ ] No hard-written weak relationships

---

## Phase E: Source Profiles + Automation

**Goal:** Scale ingestion ONLY after the Brain can reason.

### Why Last

Source Profiles feed signals into the system at scale.
Without solid:
- Evidence Pack corroboration
- Ambiguity resolution
- Relationship strengthening

...automated ingestion creates chaos, not knowledge.

### E1: Source Profile CRUD

```typescript
interface SourceProfile {
  sourceId: string;
  sourceName: string;
  sourceUrl: string;
  sourceType: 'venue_website' | 'artist_website' | 'listing_site';
  refreshSchedule: 'hourly' | 'daily' | 'weekly' | 'manual';
  ownerId?: string;
  status: 'active' | 'paused' | 'failed';
  lastFetched?: string;
  consecutiveFailures: number;
}
```

### E2: URL Rendering (Playwright)

### E3: EventBridge Scheduler

### E4: Change Detection & Deduplication

### Acceptance Criteria

- [ ] Source profiles can be created
- [ ] Manual fetch creates signals
- [ ] Scheduler runs on schedule
- [ ] Change detection prevents duplicates

---

## Success Metric

After all phases:

1. User pastes Facebook event
2. Claims generated, grouped into evidence pack
3. Ambiguity detected: "Which The Rigger?"
4. Chat asks user, user confirms
5. Event candidate ratified
6. Relationships proposed, will strengthen later
7. Second signal arrives, corroborates
8. Pack strength increases, verification promotes
9. Source profile added
10. Future signals auto-corroborate

**The Brain reasons. Then it scales.**

---

## Related

- [[now]] - Current sprint
- [[parallel-workstreams]] - Running multiple streams
- [[../05-entities/evidence-pack-model]] - Evidence pack schema
- [[../05-entities/clarification-model]] - Ambiguity resolution
- [[../05-entities/relationship-model]] - Proposed relationships
- [[../10-brain/signal-to-claim-model]] - Claim philosophy

