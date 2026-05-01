# Now

**First Vertical Slice: Evidence → Claims → Entities**

One thin path through the entire system. Manual intake first, automation later.

See [[build-plan]] for full technical breakdown.

---

## BUILD-001: Signal Inbox

**Status:** Ready
**Priority:** P0
**Build Phase:** 1

### What

Raw source capture into S3/DynamoDB.

### Supports

- Paste URL
- Paste Facebook/event text
- Upload image/screenshot
- Upload XLS/CSV gig list
- Add free-text note

### Acceptance Criteria

- [ ] `/intelligence/inbox` page with drop zone
- [ ] S3 bucket for raw files
- [ ] DynamoDB Signal table
- [ ] POST /signals Lambda
- [ ] Signal status: `received`
- [ ] Any supported content type stored

### Technical

| Component | Technology |
|-----------|------------|
| Frontend | React (bndy-backstage) |
| API | API Gateway |
| Lambda | `signal-intake` |
| Storage | S3 + DynamoDB |

---

## BUILD-002: Claim Generator

**Status:** Blocked by BUILD-001
**Priority:** P0
**Build Phase:** 2

### What

AI reads signal and outputs **claims**, not fields.

### The Prompt

Not: "Extract fields: artist, venue, date"

But: "What does this tell us about the live music world? What claims can you make? What are you uncertain about?"

### Acceptance Criteria

- [ ] Lambda triggered by new Signal
- [ ] Bedrock (Claude) interprets content
- [ ] Claims stored in DynamoDB
- [ ] Uncertainties flagged
- [ ] Signal status: `needs_review`

### Example Output

```
I think this tells us:
- Event exists: "Stingray Live"
- Artist performs: Stingray
- Venue hosts: The Rigger
- Date: 15 May 2026
- Uncertainties: year inferred, support acts unknown
```

---

## BUILD-003: Review Console

**Status:** Blocked by BUILD-002
**Priority:** P0
**Build Phase:** 3

### What

Show source left, claims right. Accept / reject / challenge.

### Layout

```
┌────────────────────────┬────────────────────────────┐
│ RAW SOURCE             │ GENERATED CLAIMS           │
├────────────────────────┼────────────────────────────┤
│ [Preview of signal]    │ ● Event: Stingray Live     │
│                        │ ● Artist: Stingray (0.94)  │
│                        │ ● Venue: The Rigger (0.91) │
│                        │ ● Date: 15 May 2026 ⚠️     │
│                        │                            │
│                        │ [Accept] [Edit] [Reject]   │
└────────────────────────┴────────────────────────────┘
```

### Acceptance Criteria

- [ ] `/intelligence/review` page
- [ ] View signal content alongside claims
- [ ] Accept individual claims
- [ ] Reject individual claims
- [ ] Challenge with reason
- [ ] Edit claim values before accepting

---

## BUILD-004: Canonical Entity Drafts

**Status:** Blocked by BUILD-003
**Priority:** P0
**Build Phase:** 4

### What

Accepted claims create/update draft canonical entities.

### Flow

```
Accepted claim: "Artist Stingray performs"
↓
Check existing: arts_123 exists?
↓
Yes → Link claim as evidence
No → Create draft entity
```

### Acceptance Criteria

- [ ] Accepted claims trigger entity resolution
- [ ] Existing entities linked (not duplicated)
- [ ] New entities created as drafts
- [ ] Relationships created (artist→event→venue)
- [ ] Draft entities can be published

---

## BUILD-005: Source Profiles

**Status:** Blocked by BUILD-004
**Priority:** P1
**Build Phase:** 5

### What

Regular URLs that generate repeat signals.

### Example

```typescript
{
  sourceName: "The Rigger website",
  sourceType: "venue_website",
  sourceUrl: "https://therigger.co.uk/whats-on",
  refreshSchedule: "daily",
  owner: "vnue_123"
}
```

### Acceptance Criteria

- [ ] Create source profile
- [ ] Manual "fetch now" button
- [ ] Fetched content becomes Signal
- [ ] Same pipeline (no special handling)

---

## BUILD-006: Automation

**Status:** Blocked by BUILD-005
**Priority:** P1
**Build Phase:** 6

### What

Scheduled fetching from source profiles.

### Acceptance Criteria

- [ ] EventBridge scheduler
- [ ] Sources fetched on schedule
- [ ] New signals created automatically
- [ ] Same claim pipeline
- [ ] Error handling and retry

---

## Success Metric

One human can:

1. Drop a Facebook event paste
2. See bndy generate claims
3. Accept/correct claims
4. See draft event created
5. See event linked to venue and artist

**Without writing any code.**

---

## Not Building Now

| Feature | Why Later |
|---------|-----------|
| Graph visualisation | Nice to have, not essential |
| Bulk import | Start manual, prove pipeline |
| Scraper framework | Source profiles first |
| Genre/location taxonomies | Derive from claims |
| Neo4j | DynamoDB relationships first |
| Migration tooling | After pipeline proven |

---

## Related

- [[build-plan]] - Full technical breakdown
- [[next]] - After vertical slice
- [[../10-brain/bndy-brain-concept|Brain Concept]] - The agentic model
- [[../10-brain/signal-to-claim-model|Signal to Claim]] - How claims work

