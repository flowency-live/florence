# Source Record Model

Tracks where data came from and how reliable it is.

## Why Sources Matter

Every event should know where it came from and how reliable it is. This turns bndy from a listing site into a **trust-aware event intelligence system**.

Without provenance:
- Can't assess data quality
- Can't debug extraction failures
- Can't prefer verified sources
- Can't explain confidence to users

## Schema

```typescript
interface SourceRecord {
  // Identity
  sourceId: string;                    // UUID

  // Classification
  sourceType: SourceType;
  sourceName: string;                  // e.g., 'dice', 'skiddle', 'venue_website'
  sourceUrl?: string;                  // Original URL

  // Capture
  capturedAt: string;                  // When we captured this
  rawText?: string;                    // Extracted text
  rawImageS3Key?: string;              // Poster/flyer image
  rawHtml?: string;                    // Original HTML (for debugging)

  // Extraction
  extractionOutput?: object;           // Structured JSON from extraction
  extractionConfidence: number;        // 0-1 how confident was extraction
  extractionMethod: ExtractionMethod;

  // Linking
  linkedEntityIds: string[];           // Events/venues/artists created from this
  linkedEntityTypes: string[];         // ['event', 'venue']

  // Quality
  processingStatus: ProcessingStatus;
  humanReviewedAt?: string;
  humanReviewedBy?: string;
  corrections?: Correction[];

  // Timestamps
  createdAt: string;
  updatedAt: string;
}

type SourceType =
  | 'api'              // Structured API (Dice, Skiddle)
  | 'website_scrape'   // Venue website
  | 'social_post'      // Instagram, Facebook post
  | 'social_event'     // Facebook event
  | 'poster_upload'    // User-uploaded poster
  | 'manual_entry'     // User typed it in
  | 'partner_feed'     // Venue/promoter direct feed
  | 'email_forward'    // Forwarded newsletter/email
  | 'migration'        // From v1 platform

type ExtractionMethod =
  | 'structured'       // Direct API/schema parsing
  | 'html_parse'       // HTML scraping
  | 'ocr_basic'        // Tesseract/basic OCR
  | 'ocr_textract'     // AWS Textract
  | 'llm_multimodal'   // Claude vision extraction
  | 'llm_text'         // Claude text extraction
  | 'manual'           // Human typed

type ProcessingStatus =
  | 'pending'          // Awaiting processing
  | 'extracted'        // AI extracted, needs review
  | 'reviewed'         // Human reviewed
  | 'published'        // Created entities
  | 'rejected'         // Spam/invalid
  | 'failed'           // Extraction failed

interface Correction {
  field: string;
  originalValue: any;
  correctedValue: any;
  correctedBy: string;
  correctedAt: string;
  reason?: string;
}
```

## Source Reliability Scoring

Track reliability per source over time:

```typescript
interface SourceReliability {
  sourceName: string;

  // Volume
  totalRecords: number;
  recordsThisMonth: number;

  // Quality
  successRate: number;           // % that became published entities
  correctionRate: number;        // % that needed human correction
  duplicateRate: number;         // % that were duplicates

  // Timeliness
  avgLeadTimeDays: number;       // How far ahead events are captured
  freshnessScore: number;        // 0-1 how up-to-date

  // Trust
  overallReliabilityScore: number;  // 0-1 computed score
}
```

## Event Provenance Fields

Every event should carry provenance:

```typescript
interface EventProvenance {
  // Primary source
  primarySourceId: string;
  primarySourceType: SourceType;

  // All sources (for corroboration)
  sourceIds: string[];

  // Confidence
  extractionConfidence: number;      // How confident was extraction
  entityResolutionConfidence: number; // How confident are venue/artist matches
  overallConfidence: number;         // Combined score

  // Verification
  verificationStatus: VerificationStatus;
  verifiedByVenue: boolean;
  verifiedByArtist: boolean;
  verifiedByTrustedContributor: boolean;
  verifiedAt?: string;
  verifiedBy?: string;

  // Corroboration
  corroboratingSourceCount: number;
  lastSeenAt: string;

  // Uncertainty
  uncertaintyFlags: UncertaintyFlag[];
}

type VerificationStatus =
  | 'unverified'
  | 'ai_extracted'
  | 'community_verified'
  | 'venue_verified'
  | 'artist_verified'
  | 'admin_verified'

type UncertaintyFlag =
  | 'date_uncertain'
  | 'time_uncertain'
  | 'venue_uncertain'
  | 'artist_uncertain'
  | 'single_source'
  | 'stale_source'
  | 'possible_cancellation'
```

## Processing Pipeline

```
Source submitted/detected
↓
SourceRecord created (status: pending)
↓
Extraction runs (status: extracted)
↓
Entity resolution matches venue/artist
↓
Human reviews if confidence < threshold (status: reviewed)
↓
Entities created/updated (status: published)
↓
SourceRecord linked to entities
```

## Admin Queues

Key queues for human review:

| Queue | Criteria |
|-------|----------|
| Low confidence extraction | extractionConfidence < 0.7 |
| Uncertain entities | entityResolutionConfidence < 0.8 |
| New sources | First record from this sourceName |
| Failed extraction | processingStatus = 'failed' |
| Spam candidates | High image noise, no text |

## Related

- [[event-model]] - Events reference source provenance
- [[venue-model]] - Venues track sources
- [[04-architecture/ingestion-pipeline|Ingestion Pipeline]]
