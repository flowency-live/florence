# Intelligence Console

The operational interface for seeing bndy's knowledge graph build.

## Purpose

The Intelligence Console is how operators and community builders see:

- What signals are coming in
- What bndy is extracting
- How entities are being resolved
- Where the graph is growing
- What needs human review

This is not an admin dashboard. It is a **knowledge growth observatory**.

## Core Views

### 1. Signal Inbox

The firehose of incoming evidence.

```
┌─────────────────────────────────────────────────────────────────────┐
│ Signal Inbox                                          Filter: All ▼ │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│ ● sgnl_abc123  │ Facebook paste │ 2 mins ago  │ Processing...       │
│ ● sgnl_def456  │ Image upload   │ 5 mins ago  │ Needs review        │
│ ✓ sgnl_ghi789  │ URL            │ 12 mins ago │ Published (3 items) │
│ ✓ sgnl_jkl012  │ Spreadsheet    │ 1 hour ago  │ Published (47 items)│
│ ✗ sgnl_mno345  │ Image          │ 2 hours ago │ Rejected (spam)     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

Filters:
- Status: Received | Processing | Needs Review | Published | Rejected
- Type: URL | Image | Text | Spreadsheet | PDF | Scraper
- Source: User submission | Email | API | Scraper
- Date range
```

### 2. Extraction Review

Side-by-side view of raw source and extracted intelligence.

```
┌──────────────────────────────┬──────────────────────────────────────┐
│ RAW SOURCE                   │ EXTRACTED ENTITIES                   │
├──────────────────────────────┼──────────────────────────────────────┤
│                              │                                      │
│ [Image: Concert poster]      │ Event: Stingray Live                 │
│                              │ ├─ Date: 15 May 2026        ✓        │
│ "STINGRAY                    │ ├─ Time: 20:00              ✓        │
│  LIVE AT THE RIGGER          │ ├─ Price: £5                ⚠️       │
│  THURSDAY 15TH MAY           │ └─ Venue: ?                          │
│  DOORS 8PM                   │                                      │
│  £5 OTD"                     │ Venue: The Rigger                    │
│                              │ ├─ City: Newcastle-under-Lyme ⚠️     │
│                              │ └─ Match: 0.94 confidence            │
│                              │                                      │
│                              │ Artist: Stingray                     │
│                              │ ├─ Genre: Rock              ⚠️       │
│                              │ └─ Match: New artist                 │
│                              │                                      │
│                              │ [Confirm All] [Edit] [Reject]        │
└──────────────────────────────┴──────────────────────────────────────┘
```

Key interactions:
- Click entity to edit fields
- Click venue match to change resolution
- Confirm to publish
- Reject to discard

### 3. Entity Resolution View

How bndy is matching names to canonical records.

```
┌─────────────────────────────────────────────────────────────────────┐
│ Entity Resolution: Venue                                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│ Input: "The Rigger"                                                 │
│                                                                     │
│ Possible matches:                                                   │
│                                                                     │
│ ┌─────────────────────────────────────────────────────────────────┐ │
│ │ ○ The Rigger                                    Confidence: 94% │ │
│ │   Newcastle-under-Lyme, ST5 1QE                                 │ │
│ │   47 events • Last event: 3 days ago                            │ │
│ │   Matched by: Name + location                                   │ │
│ └─────────────────────────────────────────────────────────────────┘ │
│                                                                     │
│ ┌─────────────────────────────────────────────────────────────────┐ │
│ │ ○ Rigger Venue                                  Confidence: 63% │ │
│ │   Stoke-on-Trent, ST1 2AB                                       │ │
│ │   12 events • Last event: 2 months ago                          │ │
│ │   Matched by: Name similarity                                   │ │
│ └─────────────────────────────────────────────────────────────────┘ │
│                                                                     │
│ ┌─────────────────────────────────────────────────────────────────┐ │
│ │ ○ Create new venue                              Confidence: 20% │ │
│ │   No existing match found                                       │ │
│ └─────────────────────────────────────────────────────────────────┘ │
│                                                                     │
│                                               [Confirm Selection]   │
└─────────────────────────────────────────────────────────────────────┘
```

### 4. Graph View

Visual representation of the relationship graph.

```
┌─────────────────────────────────────────────────────────────────────┐
│ Knowledge Graph                                    View: Local ▼    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│                    ┌─────────────┐                                  │
│                    │  The Rigger │                                  │
│                    │   (Venue)   │                                  │
│                    └──────┬──────┘                                  │
│                           │                                         │
│              hosted_at────┼────hosted_at                            │
│                   │       │       │                                 │
│            ┌──────┴──┐ ┌──┴───┐ ┌─┴───────┐                         │
│            │Event 1  │ │Event2│ │Event 3  │                         │
│            │May 15   │ │May 22│ │May 29   │                         │
│            └────┬────┘ └──┬───┘ └────┬────┘                         │
│                 │         │          │                              │
│            features    features   features                          │
│                 │         │          │                              │
│            ┌────┴────┐ ┌──┴───┐ ┌────┴────┐                         │
│            │Stingray │ │Band 2│ │ Band 3  │                         │
│            │(Artist) │ │      │ │         │                         │
│            └─────────┘ └──────┘ └─────────┘                         │
│                                                                     │
│ Legend: ○ Venue  ◇ Event  □ Artist                                  │
└─────────────────────────────────────────────────────────────────────┘

Interactions:
- Click node to see details
- Drag to pan
- Scroll to zoom
- Filter by entity type, date range, confidence
```

### 5. Knowledge Growth Dashboard

Metrics showing the graph building over time.

```
┌─────────────────────────────────────────────────────────────────────┐
│ Knowledge Growth                                   Period: 7 days ▼ │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│ ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐      │
│ │ Canonical Venues │ │ Canonical Artists│ │ Canonical Events │      │
│ │       632        │ │       618        │ │      1,089       │      │
│ │     +21 (7d)     │ │     +21 (7d)     │ │     +48 (7d)     │      │
│ └──────────────────┘ └──────────────────┘ └──────────────────┘      │
│                                                                     │
│ ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐      │
│ │ Signals Processed│ │ Sources Linked   │ │  Relationships   │      │
│ │       156        │ │       312        │ │      2,847       │      │
│ │     this week    │ │     this week    │ │     total        │      │
│ └──────────────────┘ └──────────────────┘ └──────────────────┘      │
│                                                                     │
│ ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐      │
│ │ Duplicate Merges │ │ Low Confidence   │ │  Review Queue    │      │
│ │        12        │ │        23        │ │        7         │      │
│ │     this week    │ │   items total    │ │   pending        │      │
│ └──────────────────┘ └──────────────────┘ └──────────────────┘      │
│                                                                     │
│ ──────────────────────────────────────────────────────────────────  │
│                                                                     │
│  Events over time                                                   │
│  ▄▄▄                                                                │
│  ███▄▄                                                              │
│  █████▄▄▄                                                           │
│  ████████▄▄▄▄▄                                                      │
│  ─────────────────────────────────────────────────────────────────  │
│  Apr                          May                                   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## Access Levels

| Role | Access |
|------|--------|
| Admin | Full console |
| Community Builder | Signal inbox (own signals), Entity resolution |
| Operator | All views except admin settings |
| Public | None (future: public graph explorer?) |

## Technical Implementation

### Data Sources

- **Signal Inbox**: DynamoDB Signals table, filtered by status
- **Extraction Review**: Signal + linked SourceRecords
- **Entity Resolution**: Resolution candidates from extraction Lambda
- **Graph View**: DynamoDB relationships, rendered client-side
- **Dashboard**: CloudWatch metrics + DynamoDB counts

### Frontend

- React component library
- Real-time updates via WebSocket (API Gateway)
- Graph visualization: D3.js or vis.js
- Mobile-responsive for field use

### API Endpoints

| Endpoint | Purpose |
|----------|---------|
| `GET /console/signals` | List signals with filters |
| `GET /console/signals/:id` | Signal detail with extractions |
| `POST /console/signals/:id/confirm` | Confirm extraction |
| `POST /console/signals/:id/reject` | Reject signal |
| `GET /console/resolutions` | Pending entity resolutions |
| `POST /console/resolutions/:id` | Confirm resolution |
| `GET /console/graph` | Graph data for visualization |
| `GET /console/metrics` | Dashboard metrics |

## Priority

| View | Priority | Rationale |
|------|----------|-----------|
| Signal Inbox | P0 | Must see what's coming in |
| Extraction Review | P0 | Must validate AI extractions |
| Knowledge Dashboard | P1 | Need to see growth |
| Entity Resolution | P1 | Critical for quality |
| Graph View | P2 | Nice to have, not essential initially |

## Related

- [[feature-catalogue]]
- [[../04-architecture/ingestion-pipeline|Ingestion Pipeline]]
- [[../05-entities/signal-model|Signal Model]]
- [[../05-entities/source-record-model|Source Record Model]]

