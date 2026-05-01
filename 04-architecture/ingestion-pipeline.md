# Ingestion Pipeline

How bndy turns raw evidence into canonical intelligence.

## Core Principle

Users do not create perfect records. They throw evidence into bndy.

**Evidence in. Knowledge out.**

## Universal Input: Signals

Every piece of information enters bndy as a **Signal**. There is no "add event" or "create venue". Just:

**Drop anything here. bndy will work out what it is.**

Signal types:
- URL (venue website, Facebook event, Instagram post)
- Image (poster, flyer, screenshot)
- Screenshot (gig listing, social media post)
- Text paste (Facebook copy, email forward)
- Spreadsheet (promoter gig list, venue calendar)
- PDF (press release, venue programme)
- Manual note (overheard info, phone call)
- Scraped data (automated collection)

## Pipeline Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                         SIGNAL INTAKE                                │
│   Web Upload │ Paste │ Email Forward │ API │ Scrapers                │
└─────────────────────────┬───────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         RAW STORAGE                                  │
│              S3 (files) + DynamoDB (metadata)                        │
└─────────────────────────┬───────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────────┐
│                       AI EXTRACTION                                  │
│              Bedrock (Claude) - Multi-modal                          │
│         Text │ Images │ PDFs │ Spreadsheets                          │
└─────────────────────────┬───────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     SOURCE RECORDS                                   │
│            Extracted entities with confidence scores                 │
└─────────────────────────┬───────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    ENTITY RESOLUTION                                 │
│            Match to canonical venues/artists/events                  │
└─────────────────────────┬───────────────────────────────────────────┘
                          │
              ┌───────────┴───────────┐
              │                       │
              ▼                       ▼
     ┌────────────────┐      ┌────────────────┐
     │  High Confidence│      │ Needs Review   │
     │  → Auto-publish │      │ → Human queue  │
     └────────┬───────┘      └────────┬───────┘
              │                       │
              └───────────┬───────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   CANONICAL ENTITIES                                 │
│                Venue │ Artist │ Event                                │
└─────────────────────────┬───────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   RELATIONSHIP GRAPH                                 │
│        artist_played_venue │ event_hosted_at │ similar_to            │
└─────────────────────────────────────────────────────────────────────┘
```

## Stage 1: Signal Intake

### Input Channels

| Channel | Lambda | Trigger |
|---------|--------|---------|
| Web upload | `intake-web` | API Gateway |
| Text paste | `intake-web` | API Gateway |
| Email forward | `intake-email` | SES |
| Scraper | `intake-scraper` | EventBridge Schedule |
| API | `intake-api` | API Gateway |

### User Experience

`/bndy-intake` page:

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│       Drop anything here                            │
│                                                     │
│   ┌─────────────────────────────────────────────┐   │
│   │                                             │   │
│   │     📎 Paste URL                            │   │
│   │     📷 Upload image                         │   │
│   │     📋 Paste text                           │   │
│   │     📊 Upload spreadsheet                   │   │
│   │     ✉️ Forward email                        │   │
│   │     📝 Add note                             │   │
│   │                                             │   │
│   └─────────────────────────────────────────────┘   │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Signal Storage

**S3** - Raw files:
```
s3://bndy-signals/
├── images/2026/05/01/sgnl_abc123.jpg
├── screenshots/2026/05/01/sgnl_def456.png
├── spreadsheets/2026/05/01/sgnl_ghi789.xlsx
├── pdfs/2026/05/01/sgnl_jkl012.pdf
├── html/2026/05/01/sgnl_mno345.html
└── text/2026/05/01/sgnl_pqr678.txt
```

**DynamoDB** - Signal metadata:
```typescript
Signal {
  signalId: "sgnl_abc123",
  signalType: "image",
  submittedBy: "user_xyz",
  rawS3Key: "images/2026/05/01/sgnl_abc123.jpg",
  processingStatus: "processing",
  createdAt: "2026-05-01T14:30:00Z"
}
```

## Stage 2: AI Extraction

### Multi-Modal Processing

| Input Type | Extraction Method | Model |
|------------|-------------------|-------|
| URL | Fetch + parse HTML | Claude |
| Image | Vision analysis | Claude |
| Screenshot | OCR + vision | Claude |
| Text | Direct parse | Claude |
| Spreadsheet | Parse rows → batch | Claude |
| PDF | Extract + vision | Claude |

### Extraction Prompt Pattern

```
You are analyzing a signal submitted to bndy, a grassroots music platform.

Extract any events, venues, or artists you can identify.

For each entity, provide:
- Name
- Type (event/venue/artist)
- Confidence (0.0-1.0)
- Uncertain fields

Signal type: {signalType}
Content: {content}
```

### Extraction Output

```typescript
interface ExtractionResult {
  signalId: string;
  extractedEntities: ExtractedEntity[];
  processingTime: number;
  modelUsed: string;
}

interface ExtractedEntity {
  entityType: 'event' | 'venue' | 'artist';
  name: string;
  confidence: number;
  fields: Record<string, unknown>;
  uncertainFields: string[];
}
```

### Example Extraction

**Input**: Facebook event pasted text

**Output**:
```json
{
  "extractedEntities": [
    {
      "entityType": "event",
      "name": "Stingray Live",
      "confidence": 0.85,
      "fields": {
        "date": "2026-05-15",
        "time": "20:00",
        "venueName": "The Rigger",
        "artistNames": ["Stingray"]
      },
      "uncertainFields": ["ticketPrice"]
    },
    {
      "entityType": "venue",
      "name": "The Rigger",
      "confidence": 0.82,
      "fields": {
        "city": "Newcastle-under-Lyme"
      },
      "uncertainFields": ["address", "postcode"]
    },
    {
      "entityType": "artist",
      "name": "Stingray",
      "confidence": 0.90,
      "fields": {
        "genre": "Rock"
      },
      "uncertainFields": []
    }
  ]
}
```

## Stage 3: Source Records

Each extraction creates one or more SourceRecords:

```typescript
SourceRecord {
  sourceId: "src_xxx",
  signalId: "sgnl_abc123",
  sourceType: "submission",
  sourceName: "facebook_event",
  extractionMethod: "ai",

  extractedData: { ... },
  confidence: 0.82,
  uncertainFields: ["time"],

  linkedEntityIds: [],  // Populated after resolution
  processingStatus: "pending_resolution"
}
```

## Stage 4: Entity Resolution

Match extracted entities to canonical records.

### Resolution Strategies

| Strategy | Confidence Boost | When Used |
|----------|------------------|-----------|
| Exact name match | +0.3 | Name identical |
| Fuzzy name match | +0.15 | Levenshtein < 3 |
| Location match | +0.2 | Same coordinates/postcode |
| Google Place ID | +0.4 | API confirms match |
| AI reasoning | Variable | Ambiguous cases |

### Resolution Flow

```
"The Rigger"
     │
     ▼
┌─────────────────────────────────────────┐
│ Possible matches:                       │
│                                         │
│ 1. The Rigger, Newcastle-under-Lyme     │
│    Confidence: 0.94                     │
│                                         │
│ 2. Rigger Venue, Stoke                  │
│    Confidence: 0.63                     │
│                                         │
│ 3. Create new venue                     │
│    Confidence: 0.20                     │
└─────────────────────────────────────────┘
```

### Auto-Publish Thresholds

| Confidence | Action |
|------------|--------|
| > 0.90 | Auto-publish |
| 0.70 - 0.90 | Auto-publish with review flag |
| 0.50 - 0.70 | Queue for human review |
| < 0.50 | Reject or manual only |

## Stage 5: Human Review Queue

When confidence is below threshold:

```
┌─────────────────────────────────────────────────────────────┐
│ Signal: sgnl_abc123                                          │
│ Type: Facebook paste                                         │
│ Submitted: 5 mins ago                                        │
├──────────────────────────────┬──────────────────────────────┤
│ RAW SOURCE                   │ EXTRACTED ENTITIES            │
│                              │                               │
│ "Stingray live at The        │ Event: Stingray Live          │
│  Rigger, Thursday 15th       │ ├─ Date: 15 May 2026 ✓        │
│  May. Doors 8pm.             │ ├─ Time: 20:00 ✓              │
│  £5 on the door."            │ ├─ Price: £5 ⚠️               │
│                              │ └─ Venue: The Rigger ?        │
│                              │                               │
│                              │ Venue match:                  │
│                              │ ○ The Rigger, Newcastle (0.94)│
│                              │ ○ Create new                  │
│                              │                               │
│                              │ [Confirm] [Edit] [Reject]     │
└──────────────────────────────┴──────────────────────────────┘
```

## Stage 6: Canonical Entities

Once resolved, create/update canonical records:

```typescript
CanonicalVenue {
  venueId: "vnue_xxx",
  name: "The Rigger",
  address: { ... },
  coordinates: { ... },

  sourceRecords: ["src_001", "src_002", ...],
  confidence: 0.92,
  lastUpdated: "2026-05-01T15:00:00Z"
}
```

## Stage 7: Relationship Graph

Create relationship edges:

```
ARTIST#stingray → REL#played_at → VENUE#the-rigger
EVENT#stingray-rigger-2026-05-15 → REL#hosted_at → VENUE#the-rigger
EVENT#stingray-rigger-2026-05-15 → REL#features → ARTIST#stingray
```

## Automated Scrapers

Scrapers are a special case of Signal intake:

| Source | Frequency | Signal Type |
|--------|-----------|-------------|
| Venue websites | Daily | `url` |
| Dice | Hourly | `api` |
| Skiddle | Hourly | `api` |
| Facebook | 6-hourly | `api` |
| Eventbrite | Daily | `api` |

Scrapers create Signals programmatically, then follow the same pipeline.

## Monitoring

| Metric | Alert |
|--------|-------|
| Signals received | Dashboard |
| Extraction success rate | < 85% |
| Resolution success rate | < 80% |
| Review queue depth | > 50 |
| Average processing time | > 30s |

## Related

- [[../05-entities/signal-model|Signal Model]]
- [[../05-entities/source-record-model|Source Record Model]]
- [[../02-product/intelligence-console|Intelligence Console]]
- [[aws-stack]]
- [[data-model]]

