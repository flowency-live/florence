# Signal Model

## Core Concept

A Signal is any raw input that enters bndy.

Users do not create perfect records. They throw evidence into bndy. bndy turns evidence into intelligence.

**Evidence in. Knowledge out.**

## What is a Signal?

Any of:

- URL (venue website, Facebook event, Instagram post)
- Image (poster, flyer, screenshot)
- Screenshot (gig listing, social media post)
- Facebook copy/paste (event text, page info)
- Spreadsheet (promoter gig list, venue calendar)
- PDF (press release, venue programme)
- Email forward (booking confirmation, gig announcement)
- Manual note (overheard info, phone call summary)
- Venue gig page (HTML snapshot)

## Signal vs SourceRecord

| Concept | Purpose |
|---------|---------|
| **Signal** | Raw input - exactly what was submitted |
| **SourceRecord** | Extracted intelligence - what bndy understood |

A Signal may produce:
- 0 SourceRecords (unprocessable)
- 1 SourceRecord (simple extraction)
- N SourceRecords (spreadsheet with 50 gigs)

## Signal Lifecycle

```text
Signal submitted
↓
Raw stored (S3)
↓
Metadata stored (DynamoDB)
↓
Processing queued
↓
AI extraction
↓
SourceRecord(s) created
↓
Entity resolution
↓
Canonical entities published
↓
Relationships created
```

## Signal States

| Status | Meaning |
|--------|---------|
| `received` | Just landed, not yet processed |
| `processing` | AI extraction in progress |
| `needs_review` | Extraction complete, human review needed |
| `published` | Entities created successfully |
| `rejected` | Invalid or duplicate signal |
| `failed` | Processing error |

## TypeScript Interface

```typescript
interface Signal {
  // Identity
  signalId: string;              // sgnl_xxxxxxxx

  // Submission
  signalType: SignalType;
  submittedBy: string;           // User ID or 'system'
  submittedAt: string;           // ISO 8601
  submissionMethod: SubmissionMethod;

  // Raw content
  rawS3Key?: string;             // For files/images
  rawText?: string;              // For pasted text
  sourceUrl?: string;            // For URLs

  // Processing
  processingStatus: SignalStatus;
  processedAt?: string;
  processingError?: string;

  // Outputs
  sourceRecordIds: string[];     // Created SourceRecords

  // Metadata
  metadata?: {
    fileName?: string;
    mimeType?: string;
    fileSize?: number;
    extractedTitle?: string;
  };
}

type SignalType =
  | 'url'
  | 'image'
  | 'screenshot'
  | 'text_paste'
  | 'spreadsheet'
  | 'pdf'
  | 'email'
  | 'manual_note';

type SubmissionMethod =
  | 'web_upload'
  | 'web_paste'
  | 'email_forward'
  | 'api'
  | 'scraper';

type SignalStatus =
  | 'received'
  | 'processing'
  | 'needs_review'
  | 'published'
  | 'rejected'
  | 'failed';
```

## Storage Model

### S3 Structure

```
s3://bndy-signals/
├── images/
│   └── 2026/05/01/sgnl_abc123.jpg
├── screenshots/
│   └── 2026/05/01/sgnl_def456.png
├── spreadsheets/
│   └── 2026/05/01/sgnl_ghi789.xlsx
├── pdfs/
│   └── 2026/05/01/sgnl_jkl012.pdf
├── html/
│   └── 2026/05/01/sgnl_mno345.html
└── text/
    └── 2026/05/01/sgnl_pqr678.txt
```

### DynamoDB Access Patterns

| Access Pattern | PK | SK |
|----------------|----|----|
| Get signal by ID | `SIGNAL#sgnl_xxx` | `#METADATA` |
| List signals by status | `STATUS#processing` | `SIGNAL#sgnl_xxx` (GSI) |
| List signals by submitter | `USER#user_xxx` | `SIGNAL#sgnl_xxx` (GSI) |
| List signals by date | `DATE#2026-05-01` | `SIGNAL#sgnl_xxx` (GSI) |

## Signal to SourceRecord Flow

```text
Signal: Facebook event pasted
↓
AI Extraction (Bedrock)
↓
SourceRecord {
  sourceId: "src_xxx"
  signalId: "sgnl_xxx"
  sourceType: "submission"
  sourceName: "facebook_event"
  extractionMethod: "ai"

  extractedData: {
    eventName: "Stingray Live"
    venueName: "The Rigger"
    date: "2026-05-15"
    artists: ["Stingray"]
  }

  confidence: 0.82
  uncertainFields: ["time"]
}
↓
Entity Resolution
↓
Canonical Event + Venue + Artist
```

## Extraction Confidence

| Factor | Impact |
|--------|--------|
| Clear structured text | +0.2 |
| Multiple corroborating fields | +0.15 |
| Known venue name | +0.1 |
| Ambiguous date format | -0.1 |
| Missing required fields | -0.2 |
| OCR from image | -0.1 |
| Handwritten text | -0.2 |

## ID Format

```
sgnl_[8-char-nanoid]

Examples:
sgnl_a1b2c3d4
sgnl_x9y8z7w6
```

## Related

- [[source-record-model]] - What extraction produces
- [[../04-architecture/ingestion-pipeline|Ingestion Pipeline]] - Processing flow
- [[../02-product/intelligence-console|Intelligence Console]] - Review UI

