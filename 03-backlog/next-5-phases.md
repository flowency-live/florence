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

| Phase | Name | Status | Why This Order |
|-------|------|--------|----------------|
| A | Event Aggregation | ✅ Complete | Complete the candidate → entity cycle |
| B | Evidence Packs & Corroboration | Next | The central cognitive structure |
| C | Ambiguity Resolution | Planned | First-class ambiguity model, chat as primary interface |
| D | Proposed Relationships | Planned | Graph emerges from evidence, not hard-written |
| E | Source Profiles + Automation | Planned | Scale ingestion ONLY after reasoning is solid |

---

## Terminology

| Legacy | Brain-Native |
|--------|--------------|
| Draft entity | Candidate entity, Proposed entity |
| Create entity | Entity emerges from evidence |
| Auto-create relationship | Proposed relationship strengthens with corroboration |
| Review UI | Conversational ratification |

---

## Phase A: Event Aggregation ✅

**Goal:** Candidate events emerge from signal interpretation.
**Status:** Complete (2026-05-04)

### AI-Native Approach

**Critical insight:** The LLM proposes event candidates directly during interpretation (AI-native), NOT deterministic code aggregating claims after the fact (legacy ETL).

```
LLM does:
├── Interprets signal content
├── Proposes eventCandidates with reasoning
├── Explains uncertainty/ambiguity
└── Identifies clarificationQuestions

Code does:
├── Validates schema compliance
├── Persists proposals to DynamoDB
├── Resolves entities (venue name → venueId)
├── Creates clarification requests
└── Prevents unsafe mutation
```

### A1: Event Candidate Schema

See [[../05-entities/event-candidate-model]] for full schema.

```typescript
interface EventCandidate {
  candidateId: string;           // cand_xxxxxxxx
  candidateType: 'event';
  signalId: string;
  interpretationId: string;

  // LLM-proposed fields
  proposedName: string;
  proposedDate?: string;
  proposedTime?: string;
  proposedVenueId?: string;      // Entity-resolved
  proposedArtistIds: string[];   // Entity-resolved

  // Evidence chain
  sourceClaims: ClaimReference[];

  // Completeness
  completeness: 'partial' | 'complete';
  missingFields: string[];
  ambiguities: Ambiguity[];

  // Trust level
  verificationStatus: 'unverified' | 'submitter_verified';

  status: 'proposed' | 'ratified' | 'rejected' | 'merged';
}
```

### A2: Interpretation-Runner Updates

The interpretation-runner now outputs:
- `claims` - Fine-grained facts (existing)
- `eventCandidates` - AI-proposed events (new)
- `clarificationQuestions` - Ambiguity flags (new)

Entity resolution happens during interpretation:
- `proposedVenueName: "The Rigger"` → code finds `vnue_abc12345`
- Multiple matches → ambiguity added to candidate
- No match → will be created on ratification

### A3: Event Candidate API

```
GET  /candidates              - List proposed candidates
GET  /candidates/{id}         - Get candidate details
POST /candidates/{id}/ratify  - Create canonical event
POST /candidates/{id}/reject  - Reject with reason
```

Ratification creates canonical event with:
- Venue and artist entity links
- Evidence chain back to signal/claims

### A4: Claim Aggregator (Fallback)

Kept as fallback for low-confidence cases where LLM outputs claims but doesn't propose candidates. Deterministic grouping based on claim types.

### Acceptance Criteria

- [x] Event candidates proposed during interpretation (AI-native)
- [x] Venue/artist names resolved to entity IDs
- [x] Ambiguities tracked when multiple matches
- [x] Candidates can be ratified via API
- [x] Ratified candidates create canonical events
- [x] CDK stack updated with new Lambda and routes

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

