# Next 5 Phases

**Last updated:** 2026-05-03

The vertical slice is complete. Now we expand capabilities.

---

## Current State (Foundation Complete)

| Build | Name | Status |
|-------|------|--------|
| BUILD-001 | Signal Inbox | ✅ Deployed |
| BUILD-002 | Claim Generator | ✅ Deployed |
| BUILD-003 | Review Console | ✅ Deployed |
| BUILD-004 | Canonical Entity Drafts | ✅ Deployed |
| Chat UI | Conversational submission | ✅ Deployed |

**What Works:**
- Drop/paste text or image → claims generated
- Accept/reject/challenge claims
- Accepted claims → draft entities (artist, venue)
- Fuzzy name matching with normalization
- 69 tests passing

**What's Missing:**
- Events not auto-created (require aggregation)
- Entity relationships (artist→event→venue)
- Entity publishing workflow
- Source profiles (automated fetching)
- Scheduled automation

---

## Phase A: Event Aggregation

**Goal:** Complete the claim → entity cycle by enabling event creation from aggregated claims.

### A1: Event Candidate Generator

When multiple related claims exist (event_exists + event_date + venue_hosts + artist_performs), propose an event entity.

```
Claims:
├── event_exists: "Stingray Live"
├── event_date: "2026-05-15"
├── event_time: "20:00"
├── venue_hosts: "The Rigger" → vnue_abc123
└── artist_performs: "Stingray" → arts_xyz789

↓

Event Candidate:
├── name: "Stingray Live"
├── startDate: "2026-05-15"
├── startTime: "20:00"
├── venueId: vnue_abc123
└── artistIds: [arts_xyz789]
```

**Files:**
- `functions/event-aggregator/index.ts`
- `functions/shared/entities/event-candidate.ts`

**Acceptance Criteria:**
- [ ] Related claims grouped by signal/interpretation
- [ ] Event candidates proposed when enough data
- [ ] Venue and artist IDs linked (not names)
- [ ] Candidates appear in review queue

### A2: Event Ratification

Human confirms/rejects event candidates via chat or UI.

**Acceptance Criteria:**
- [ ] Review event candidates
- [ ] Confirm → creates CanonicalEvent (draft)
- [ ] Reject with reason
- [ ] Edit before confirming (date, time, venue)

### A3: Entity Publishing

Move entities from draft → published.

**Acceptance Criteria:**
- [ ] Publish individual entities
- [ ] Bulk publish from review queue
- [ ] Published events visible in API
- [ ] Published entities have publishedAt timestamp

---

## Phase B: Source Profiles

**Goal:** Enable repeated signal generation from known URLs.

### B1: Source Profile CRUD

```typescript
interface SourceProfile {
  sourceId: string;           // src_xxxxxxxx
  sourceName: string;         // "The Rigger Website"
  sourceType: 'venue_website' | 'artist_website' | 'listing_site' | 'facebook_page';
  sourceUrl: string;          // "https://therigger.co.uk/whats-on"
  refreshSchedule: 'hourly' | 'daily' | 'weekly' | 'manual';
  ownerId?: string;           // vnue_123 or arts_456
  status: 'active' | 'paused' | 'failed';
  lastFetched?: string;
  lastSignalId?: string;
  consecutiveFailures: number;
  createdAt: string;
  updatedAt: string;
}
```

**API:**
```
POST /sources              → Create source profile
GET /sources               → List sources
GET /sources/{id}          → Get source
PATCH /sources/{id}        → Update source
DELETE /sources/{id}       → Delete source
POST /sources/{id}/fetch   → Manual fetch now
```

**Acceptance Criteria:**
- [ ] Source profile schema and Lambda
- [ ] CRUD API endpoints
- [ ] Link source to owner entity (venue/artist)

### B2: Manual Fetch

"Fetch now" button triggers signal creation from source URL.

**Flow:**
```
Source Profile (src_rigger)
↓
Fetch URL content
↓
Create Signal (same pipeline)
↓
Claims generated
↓
Review queue
```

**Acceptance Criteria:**
- [ ] Fetch endpoint triggers URL render
- [ ] Content stored as signal
- [ ] Same interpretation pipeline
- [ ] Source profile updated (lastFetched, lastSignalId)

### B3: URL Rendering

Render URLs with Playwright for dynamic content.

**Acceptance Criteria:**
- [ ] Lambda or ECS task with Playwright
- [ ] Extract visible text, DOM, screenshot
- [ ] Handle SPAs and lazy-loaded content
- [ ] Timeout and error handling

---

## Phase C: Automation

**Goal:** Scheduled fetching without human intervention.

### C1: EventBridge Scheduler

Trigger source fetches on schedule.

**Acceptance Criteria:**
- [ ] EventBridge rule per refresh schedule
- [ ] Lambda processes due sources
- [ ] Rate limiting (max concurrent fetches)
- [ ] Backoff on failures

### C2: Change Detection

Only create signals when content changes.

**Acceptance Criteria:**
- [ ] Hash content on fetch
- [ ] Compare to previous hash
- [ ] Skip signal creation if unchanged
- [ ] Track change history

### C3: Failure Handling

Handle fetch failures gracefully.

**Acceptance Criteria:**
- [ ] Track consecutive failures
- [ ] Auto-pause after N failures
- [ ] Alert on repeated failures
- [ ] Resume mechanism

---

## Phase D: Relationships

**Goal:** Build the relationship graph from entities.

### D1: Relationship Schema

```typescript
interface Relationship {
  relationshipId: string;     // rel_xxxxxxxx
  fromEntityId: string;
  toEntityId: string;
  relationshipType: 'performed_at' | 'hosted' | 'booked_for' | 'similar_to';
  strength: 'weak' | 'moderate' | 'strong';
  evidence: EvidenceLink[];
  firstSeen: string;
  lastSeen: string;
  occurrenceCount: number;
  createdAt: string;
  updatedAt: string;
}
```

**Acceptance Criteria:**
- [ ] Relationship schema in shared/entities
- [ ] Store in DynamoDB with GSI for queries
- [ ] API to list relationships for entity

### D2: Auto-Relationship Creation

When an event is created, auto-generate relationships.

```
Event: evnt_123
├── venueId: vnue_456
└── artistIds: [arts_789, arts_abc]

↓ Auto-create:

arts_789 --performed_at--> vnue_456
arts_abc --performed_at--> vnue_456
vnue_456 --hosted--> arts_789
vnue_456 --hosted--> arts_abc
```

**Acceptance Criteria:**
- [ ] Relationships created on event publish
- [ ] Update existing relationships (increment occurrenceCount)
- [ ] Track firstSeen/lastSeen

### D3: Relationship Queries

API to query relationship data.

**API:**
```
GET /artists/{id}/venues       → Venues this artist has played
GET /venues/{id}/artists       → Artists this venue has hosted
GET /artists/{id}/similar      → Similar artists (future)
```

**Acceptance Criteria:**
- [ ] Query relationships by entity
- [ ] Include strength and occurrence data
- [ ] Sort by recency or frequency

---

## Phase E: Trust & Verification

**Goal:** Build trust levels for entities and claims.

### E1: Evidence Packs

Group claims that corroborate each other.

```typescript
interface EvidencePack {
  packId: string;
  signals: string[];           // Signal IDs
  claims: string[];            // Claim IDs
  corroborationStrength: 'weak' | 'moderate' | 'strong';
  proposedEntities: string[];  // Entity IDs
  status: 'gathering' | 'ready' | 'applied';
}
```

**Acceptance Criteria:**
- [ ] Evidence pack schema
- [ ] Group related signals/claims
- [ ] Calculate corroboration strength

### E2: Verification Levels

Track how verified each entity is.

```
unverified → submitter_verified → community_verified → source_correlated → owner_confirmed
```

**Acceptance Criteria:**
- [ ] Verification status on entities
- [ ] Promote verification when corroborated
- [ ] Show verification badges in UI

### E3: Claim Lineage

Track where claims came from and how they evolved.

**Acceptance Criteria:**
- [ ] Claim references source signal
- [ ] Track edits and challenges
- [ ] Show claim history in review

---

## Implementation Order

```
Phase A (Week 1-2)
├── A1: Event aggregator
├── A2: Event ratification
└── A3: Entity publishing

Phase B (Week 2-3)
├── B1: Source profile CRUD
├── B2: Manual fetch
└── B3: URL rendering (may be complex)

Phase C (Week 3-4)
├── C1: EventBridge scheduler
├── C2: Change detection
└── C3: Failure handling

Phase D (Week 4-5)
├── D1: Relationship schema
├── D2: Auto-relationship creation
└── D3: Relationship queries

Phase E (Week 5-6)
├── E1: Evidence packs
├── E2: Verification levels
└── E3: Claim lineage
```

---

## Parallel Opportunities

| Stream | Can Run With | Notes |
|--------|--------------|-------|
| A1-A3 (Events) | B1 (Source CRUD) | Different repos/concerns |
| B2-B3 (Fetch) | D1 (Relationships) | Different repos |
| C1-C3 (Automation) | D2-D3 (Auto-relationships) | After both B and D foundations |

See [[parallel-workstreams]] for how to run multiple sessions.

---

## Success Metrics

After Phase E:

1. User drops Facebook event
2. Claims generated and reviewed
3. Event created with venue and artist links
4. Relationships auto-created
5. Source profile added for venue website
6. Next week: new events auto-fetched
7. Multiple sources corroborate → higher verification

**The system learns and improves without code changes.**

---

## Related

- [[now]] - Current sprint
- [[parallel-workstreams]] - Running multiple streams
- [[../05-entities/canonical-entity-model]] - Entity schemas
- [[../05-entities/relationship-model]] - Relationship design
- [[../10-brain/signal-to-claim-model]] - Claim philosophy

