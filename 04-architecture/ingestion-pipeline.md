# Ingestion Pipeline

How bndy collects event data from external sources.

## Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                    EventBridge Scheduler                          │
│                    (cron triggers)                                 │
└─────────────────────────┬────────────────────────────────────────┘
                          │
                          ▼
┌──────────────────────────────────────────────────────────────────┐
│                     Source Scrapers                               │
│         (Lambda per source: Facebook, Dice, Skiddle, etc.)        │
└─────────────────────────┬────────────────────────────────────────┘
                          │
                          ▼
┌──────────────────────────────────────────────────────────────────┐
│                      Raw Data Queue                               │
│                         (SQS)                                     │
└─────────────────────────┬────────────────────────────────────────┘
                          │
                          ▼
┌──────────────────────────────────────────────────────────────────┐
│                    Normalisation                                  │
│            (Lambda: parse to canonical schema)                    │
└─────────────────────────┬────────────────────────────────────────┘
                          │
                          ▼
┌──────────────────────────────────────────────────────────────────┐
│                  Entity Resolution                                │
│         (Lambda: match venue/artist to canonical ID)              │
└─────────────────────────┬────────────────────────────────────────┘
                          │
                          ▼
┌──────────────────────────────────────────────────────────────────┐
│                 Duplicate Detection                               │
│            (Lambda: check for existing event)                     │
└─────────────────────────┬────────────────────────────────────────┘
                          │
              ┌───────────┴───────────┐
              │                       │
              ▼                       ▼
       ┌─────────────┐         ┌─────────────┐
       │   New Event │         │  Duplicate  │
       │   → Save    │         │  → Merge    │
       └─────────────┘         └─────────────┘
```

## Pipeline Stages

### 1. Scheduling

EventBridge Scheduler triggers scrapers on schedule:

| Source | Frequency | Notes |
|--------|-----------|-------|
| High-volume (Dice, Skiddle) | Hourly | Lots of events |
| Medium (venue websites) | Daily | Less frequent updates |
| Social (Facebook, Instagram) | 6-hourly | API rate limits |

### 2. Source Scrapers

Each source has a dedicated Lambda:

```typescript
interface ScraperOutput {
  source: string;
  rawEvents: RawEvent[];
  errors: ScraperError[];
  stats: {
    fetched: number;
    parsed: number;
    failed: number;
  };
}
```

Scrapers are responsible for:
- Fetching data from source
- Basic parsing (extract event data)
- Error handling and retry logic
- Outputting to raw data queue

### 3. Normalisation

Transform raw event to canonical schema:

```typescript
// Input: raw event from scraper
{
  "title": "Jazz Night @ The Blue Note",
  "venue_name": "Blue Note Jazz Club",
  "date": "15/06/2025",
  "time": "8pm",
  "price": "£10"
}

// Output: normalised event
{
  "name": "Jazz Night",
  "venueName": "Blue Note Jazz Club",  // Not yet resolved
  "startDate": "2025-06-15",
  "startTime": "20:00",
  "price": { "currency": "GBP", "amount": 10, "isFree": false }
}
```

### 4. Entity Resolution

Match venue/artist names to canonical IDs:

```typescript
interface ResolutionResult {
  entityType: 'venue' | 'artist';
  inputName: string;

  // Match result
  matchedId?: string;
  confidence: number;

  // If no match
  suggestedAction?: 'create' | 'review';
}
```

Resolution strategies:
1. **Exact match** - Name matches existing entity
2. **Fuzzy match** - High similarity to existing (Levenshtein, phonetic)
3. **Location match** - For venues, same address/coordinates
4. **AI match** - LLM-based reasoning for ambiguous cases

Confidence thresholds:
- `> 0.95` - Auto-link
- `0.7 - 0.95` - Link with low confidence flag
- `< 0.7` - Queue for human review or create new

### 5. Duplicate Detection

Check if event already exists:

```typescript
interface DuplicateCheck {
  isLikelyDuplicate: boolean;
  existingEventId?: string;
  confidence: number;
  reason: string;
}
```

Duplicate signals:
- Same venue + same date + similar name
- Same source ID (re-import)
- Overlapping artists and time

If duplicate:
- Merge source references
- Update fields if new data is fresher
- Don't create new event

### 6. Storage

Save to DynamoDB with:
- Full event record
- Source references
- Confidence scores
- Timestamps

Emit EventBridge event for:
- New event created
- Event updated
- Review required

## Error Handling

| Error Type | Response |
|------------|----------|
| Source unavailable | Retry with backoff, alert if persistent |
| Parse failure | Log raw data, skip event, alert if > 10% |
| Resolution failure | Queue for human review |
| Storage failure | Retry, dead-letter queue |

## Monitoring

| Metric | Alert Threshold |
|--------|-----------------|
| Ingestion success rate | < 90% |
| Source errors | > 5 per run |
| Resolution queue depth | > 100 items |
| Duplicate rate | > 50% (might indicate stale scraping) |

## Initial Sources

| Source | Type | Priority | Notes |
|--------|------|----------|-------|
| Dice | API | P0 | Good coverage, clean data |
| Skiddle | API | P0 | Wide UK coverage |
| Venue websites | Scrape | P1 | Direct source, varies by venue |
| Facebook Events | API | P2 | Rate limited, auth required |
| Eventbrite | API | P1 | Ticketed events |

## Related

- [[aws-stack]]
- [[data-model]]
