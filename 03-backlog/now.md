# Now

**First Vertical Slice: Evidence → Claims → Entities**

One thin path through the entire system. Manual intake first, automation later.

See [[build-plan]] for full technical breakdown.

---

## BUILD-001: Signal Inbox

**Status:** ✅ DEPLOYED
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
- [x] S3 bucket for raw files (`bndy-signals-dev-771551874768`)
- [x] DynamoDB Signal table (`bndy-signals-dev`)
- [x] POST /signals Lambda (`bndy-signals-intake-dev`)
- [x] Signal status: `received`
- [x] Any supported content type stored

### Technical

| Component | Technology | Status |
|-----------|------------|--------|
| Frontend | React (bndy-frontstage) | ✅ /dropzone with drag-drop, clipboard paste, text |
| Frontend | React (bndy-frontstage) | ✅ /chat conversational gig submission |
| API | API Gateway | ✅ Deployed |
| Lambda | `signal-intake` | ✅ Supports text + image |
| Lambda | `deterministic-extractor` | ✅ Textract OCR for images |
| Storage | S3 + DynamoDB | ✅ Deployed |

### API

```
POST https://9tq7w39hb2.execute-api.eu-west-2.amazonaws.com/dev/signals
GET  https://9tq7w39hb2.execute-api.eu-west-2.amazonaws.com/dev/signals/{signalId}
```

---

## BUILD-002: Claim Generator

**Status:** ✅ DEPLOYED
**Priority:** P0
**Build Phase:** 2

### What

AI reads signal and outputs **claims**, not fields.

### The Prompt

Not: "Extract fields: artist, venue, date"

But: "What does this tell us about the live music world? What claims can you make? What are you uncertain about?"

### Acceptance Criteria

- [x] Lambda triggered by new Signal (Step Functions workflow)
- [x] Bedrock (Claude Haiku 4.5) interprets content
- [x] Claims stored in DynamoDB
- [x] Uncertainties flagged
- [x] Signal status: `pending_review`

### Technical

| Component | Status |
|-----------|--------|
| Step Functions workflow | ✅ `bndy-signals-workflow-dev` |
| Deterministic extractor | ✅ Textract OCR for images |
| Interpretation runner | ✅ `bndy-signals-interpreter-dev` |
| Failure handler | ✅ `bndy-signals-failure-handler-dev` |
| Dead letter queue | ✅ `bndy-signals-failed-dev` |
| Bedrock model | ✅ `eu.anthropic.claude-haiku-4-5-20251001-v1:0` |

### Example Output

Tested with: `"STINGRAY LIVE AT THE RIGGER THURSDAY 15TH MAY 8PM"`

```json
{
  "summary": "Announcement for Stingray performing at The Rigger on Thursday, May 15th at 8PM",
  "claims": [
    { "type": "event_exists", "subject": "Stingray Live at The Rigger", "strength": "moderate" },
    { "type": "artist_performs", "subject": "Stingray", "object": "The Rigger", "strength": "moderate" },
    { "type": "venue_hosts", "subject": "The Rigger", "object": "Stingray Live", "strength": "moderate" },
    { "type": "event_date", "subject": "Stingray Live", "value": "2026-05-15", "strength": "weak" },
    { "type": "event_time", "subject": "Stingray Live", "value": "20:00", "strength": "moderate" }
  ],
  "uncertainties": ["Year not specified - inferred from current date", "Venue location unknown"]
}
```

**Note:** Times are always event start times (no "doors" concept for grassroots venues).

### Cost Tracking

```
tokensIn: 1032, tokensOut: 689, modelCost: $0.0036, runtimeMs: 3706
```

---

## BUILD-003: Review Console

**Status:** ✅ DEPLOYED (in /dropzone)
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

### API

```
POST /signals/{signalId}/claims/{claimId}/review
Body: { action: "accept" | "reject" | "challenge", reason?: string, editedObject?: string, editedValue?: string }
```

### Acceptance Criteria

- [x] Review endpoint deployed (`bndy-signals-claim-review-dev`)
- [x] Accept individual claims (in /dropzone UI)
- [x] Reject individual claims
- [x] Challenge with reason
- [x] Claim status updates in UI
- [ ] Edit claim values before accepting (API supports, UI pending)

---

## BUILD-004: Canonical Entity Drafts

**Status:** ✅ DEPLOYED
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

- [x] Accepted claims trigger entity resolution
- [x] Existing entities linked (not duplicated)
- [x] New entities created as drafts
- [ ] Relationships created (artist→event→venue)
- [ ] Draft entities can be published

### Technical

| Component | Status |
|-----------|--------|
| Canonical entity schemas | ✅ `canonical-entity.ts` |
| Entity resolver | ✅ `entity-resolver/index.ts` |
| Claim-review integration | ✅ Calls resolver on accept |
| Fuzzy name matching | ✅ Levenshtein, threshold 0.85 |
| Tests | ✅ 69 tests passing |

---

## BUILD-005: Source Profiles

**Status:** ⏳ NEXT (unblocked)
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

- [[next-5-phases]] - Next 5 implementation phases (Phase A-E)
- [[build-plan]] - Full technical breakdown
- [[parallel-workstreams]] - Running multiple build streams
- [[next]] - After vertical slice
- [[../10-brain/bndy-brain-concept|Brain Concept]] - The agentic model
- [[../10-brain/signal-to-claim-model|Signal to Claim]] - How claims work

