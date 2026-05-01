# Build Plan: First Vertical Slice

## The Goal

One thin vertical slice:

```
Drop source
→ store raw evidence
→ AI generates claims
→ human reviews claims
→ accepted claims create/update draft canonical entities
```

## What We're NOT Building First

- Full graph visualisation
- Automated scraping framework
- Genre taxonomy
- Location model perfection
- Neo4j/graph DB
- Bulk migration tooling

## Build Order

### Phase 1: Signal Inbox

Raw source capture into S3/DynamoDB.

**Supports:**
- Paste URL
- Paste Facebook/event text
- Upload image/screenshot
- Upload XLS/CSV gig list
- Add free-text note

**Stores everything as a Signal.**

| Component | Purpose |
|-----------|---------|
| S3 bucket | Raw files (images, PDFs, spreadsheets) |
| DynamoDB table | Signal metadata |
| Lambda | Intake handler |
| API Gateway | POST /signals |
| Frontend | `/intelligence/inbox` page |

**Acceptance criteria:**
- [ ] Drop any supported content
- [ ] Signal stored with unique ID
- [ ] Raw content preserved in S3
- [ ] Metadata in DynamoDB
- [ ] Status: `received`

---

### Phase 2: Signal Reader

Render, extract, then generate claims. See [[../04-architecture/signal-reader|Signal Reader]] for full architecture.

**For URL signals:**
```
URL signal
→ Playwright render (headless browser)
→ Extract: visible text + DOM + screenshot + image URLs
→ Claim generation
```

**For image signals:**
```
Image signal
→ Preprocess (resize, normalize)
→ OCR + vision model
→ Claim generation
```

**For text signals:**
```
Text signal
→ Clean/normalize
→ Claim generation
```

| Component | Purpose |
|-----------|---------|
| Lambda | Signal reader orchestration |
| Playwright | URL rendering |
| Claim generator | Interpret content, output claims |
| DynamoDB | Claim storage |

**The prompt is not:**
> "Extract fields: artist, venue, date"

**The prompt is:**
> "What does this tell us about the live music world? What claims can you make? What are you uncertain about?"

**Output example:**
```
I think this tells us:
- Event exists
- Artist "Stingray" performs
- Venue "The Rigger" hosts
- Date: 15 May 2026
- Time: 20:00 (doors)
- Ticket source: Gigantic
- Uncertainties: year inferred, exact address unknown
```

**Acceptance criteria:**
- [ ] URL signals rendered via Playwright
- [ ] Image signals preprocessed + OCR
- [ ] Text signals cleaned
- [ ] Claims stored with signal reference
- [ ] Uncertainties flagged
- [ ] Status: `needs_review`

---

### Phase 3: Review Console

Show source left, claims right. Accept / reject / challenge.

| Component | Purpose |
|-----------|---------|
| Frontend | `/intelligence/review` page |
| API | GET/POST claim actions |

**Layout:**
```
┌──────────────────────────┬──────────────────────────────┐
│ RAW SOURCE               │ GENERATED CLAIMS             │
├──────────────────────────┼──────────────────────────────┤
│                          │                              │
│ [Image/text/URL preview] │ ● Event exists               │
│                          │   → Stingray @ The Rigger    │
│                          │                              │
│                          │ ● Artist: Stingray           │
│                          │   → Match: arts_123 (0.94)   │
│                          │                              │
│                          │ ● Venue: The Rigger          │
│                          │   → Match: vnue_456 (0.91)   │
│                          │                              │
│                          │ ● Date: 15 May 2026          │
│                          │   ⚠️ Year inferred           │
│                          │                              │
│                          │ [Accept All] [Edit] [Reject] │
└──────────────────────────┴──────────────────────────────┘
```

**Acceptance criteria:**
- [ ] View signal alongside claims
- [ ] Accept individual claims
- [ ] Reject individual claims
- [ ] Challenge with reason
- [ ] Edit claim values before accepting

---

### Phase 4: Canonical Entity Drafts

Accepted claims propose artist/venue/event updates.

| Component | Purpose |
|-----------|---------|
| Lambda | Entity resolution |
| DynamoDB | Draft canonical entities |

**Flow:**
```
Accepted claim: "Artist Stingray performs"
↓
Check existing: arts_123 exists?
↓
Yes → Link claim as evidence
No → Create draft entity
↓
Draft entity awaits confirmation or auto-publishes
```

**Acceptance criteria:**
- [ ] Accepted claims create entity proposals
- [ ] Existing entities linked (not duplicated)
- [ ] New entities created as drafts
- [ ] Relationships created (artist→event→venue)

---

### Phase 5: Source Profiles

Regular URLs that generate repeat signals.

**Source Profile example:**
```typescript
{
  sourceId: "src_rigger_website",
  sourceName: "The Rigger website",
  sourceType: "venue_website",
  sourceUrl: "https://therigger.co.uk/whats-on",
  refreshSchedule: "daily",
  owner: "vnue_123",
  lastFetched: "2026-05-01T00:00:00Z"
}
```

**Acceptance criteria:**
- [ ] Create source profile
- [ ] Manual "fetch now" button
- [ ] Fetched content becomes Signal
- [ ] Same pipeline (no special handling)

---

### Phase 6: Automation

Scheduled fetching/scraping/parsing.

| Component | Purpose |
|-----------|---------|
| EventBridge | Scheduler |
| Lambda | Fetch sources by schedule |
| Source Profiles | What to fetch |

**Acceptance criteria:**
- [ ] Sources fetched on schedule
- [ ] New signals created automatically
- [ ] Same claim generation pipeline
- [ ] Errors logged, retried

---

## First Useful MVP

A page at:

```
/intelligence/inbox
```

With:

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│              Drop anything here                         │
│                                                         │
│  ┌───────────────────────────────────────────────────┐  │
│  │                                                   │  │
│  │     📎 Paste URL                                  │  │
│  │     📋 Paste text (Facebook, email, etc.)         │  │
│  │     📷 Upload image/screenshot                    │  │
│  │     📊 Upload spreadsheet                         │  │
│  │     📝 Add note                                   │  │
│  │                                                   │  │
│  └───────────────────────────────────────────────────┘  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Prove it with one real example:**

1. Facebook event text
2. Gigantic URL
3. Screenshot of poster

bndy responds:

> I think this tells us:
> - Event exists: "Stingray Live"
> - Artist performs: Stingray
> - Venue hosts: The Rigger, Newcastle-under-Lyme
> - Date: 15 May 2026
> - Time: Doors 8pm
> - Ticket source: Gigantic confirms
> - Uncertainties: Support acts not listed

**That is the foundation.**

---

## Technical Stack

| Layer | Technology |
|-------|------------|
| Frontend | React (existing bndy-backstage) |
| API | API Gateway + Lambda |
| AI | Bedrock (Claude Sonnet) |
| Storage | S3 (raw) + DynamoDB (metadata) |
| Auth | Existing Cognito |

---

## Not in Scope for V1

| Feature | Why Later |
|---------|-----------|
| Graph visualisation | Nice to have, not essential |
| Bulk import | Start manual, prove pipeline |
| Scraper framework | Source profiles first |
| Genre/location taxonomies | Derive from claims later |
| Neo4j | DynamoDB relationships first |
| Migration tooling | After pipeline proven |

---

## Success Metric

One human can:

1. Drop a Facebook event paste
2. See bndy generate claims
3. Accept/correct claims
4. See draft event created
5. See event linked to venue and artist

Without writing any code.

That's the vertical slice.

---

## Related

- [[now]] - Current backlog items
- [[../10-brain/bndy-brain-concept|Brain Concept]] - The agentic model
- [[../10-brain/signal-to-claim-model|Signal to Claim]] - How claims work
- [[../10-brain/agentic-intake-loop|Agentic Intake Loop]] - The reasoning cycle
- [[../02-product/intelligence-console|Intelligence Console]] - UI spec

