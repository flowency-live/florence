# Event Model

The core discovery unit - what's happening, where, and when.

## Why Events Matter

Events are what users actually attend:
- Discovery starts with "what's on tonight?"
- Events connect venues and artists
- Events have urgency (date-bound)

## Core Challenges

### Temporality

Events are ephemeral:
- Future events need promotion
- Past events become historical data
- Cancelled/postponed events need handling

### Completeness

Event data is often incomplete:
- No artist listed
- Vague times ("doors 7pm")
- Missing ticket links
- No genre/vibe info

### Duplication

Same event from multiple sources:
- Venue website + Dice + Facebook
- Slightly different names/times
- Need to merge, not duplicate

## Schema

```typescript
interface Event {
  // Identity
  id: string;                      // Canonical UUID
  name: string;                    // Event title
  slug: string;                    // URL-safe identifier

  // When
  startDate: string;               // "2025-06-15" (ISO)
  startTime?: string;              // "20:00" (24hr)
  endDate?: string;                // For multi-day
  endTime?: string;
  doorsTime?: string;              // When doors open
  timezone: string;                // "Europe/London"

  // Where
  venueId: string;                 // Canonical venue reference
  venue?: VenueSnapshot;           // Denormalised for display

  // Who
  artists: EventArtist[];          // Performing artists

  // What
  description?: string;
  genres: string[];
  eventType: EventType;
  ageRestriction?: '18+' | '21+' | 'all_ages';

  // Tickets
  ticketUrl?: string;
  infoUrl?: string;
  pricing?: EventPricing;

  // Media
  imageUrl?: string;
  images?: string[];

  // Status
  status: EventStatus;
  cancelledAt?: string;
  postponedTo?: string;            // New event ID if rescheduled

  // Quality
  confidenceScore: number;
  sources: SourceReference[];

  // Engagement (future)
  saveCount?: number;
  viewCount?: number;

  // Timestamps
  createdAt: string;
  updatedAt: string;
}

interface EventArtist {
  artistId: string;
  name: string;                    // Denormalised for display
  role: 'headliner' | 'support' | 'resident' | 'special_guest' | 'tba';
  billingOrder: number;            // 1 = top of bill
}

interface VenueSnapshot {
  id: string;
  name: string;
  city: string;
  coordinates?: { lat: number; lng: number };
}

interface EventPricing {
  isFree: boolean;
  currency?: string;               // "GBP"
  minPrice?: number;               // Advance price
  maxPrice?: number;               // Door price
  priceNote?: string;              // "£10 advance / £12 door"
}

type EventType =
  | 'gig'
  | 'club_night'
  | 'festival'
  | 'open_mic'
  | 'acoustic'
  | 'dj_set'
  | 'jam_session'
  | 'album_launch'
  | 'residency'
  | 'other';

type EventStatus =
  | 'confirmed'
  | 'tentative'
  | 'cancelled'
  | 'postponed'
  | 'sold_out'
  | 'past';
```

## Lifecycle

```
         ┌──────────────────────────────────────┐
         │                                      │
         ▼                                      │
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Tentative  │───►│  Confirmed  │───►│    Past     │
└─────────────┘    └──────┬──────┘    └─────────────┘
                          │
              ┌───────────┼───────────┐
              │           │           │
              ▼           ▼           ▼
       ┌──────────┐ ┌──────────┐ ┌──────────┐
       │ Cancelled│ │ Postponed│ │ Sold Out │
       └──────────┘ └──────────┘ └──────────┘
```

## Duplicate Detection

Events are likely duplicates if:

| Signal | Weight | Notes |
|--------|--------|-------|
| Same venue + same date | High | Core match |
| Similar name (> 0.8) | Medium | Different phrasing |
| Same source ID | Definitive | Re-import |
| Overlapping artists | Medium | Support acts vary |
| Same ticket URL | High | Definitive |

### Merge Strategy

1. **Keep earliest-created ID** as canonical
2. **Merge artists** (union, avoid duplicates)
3. **Prefer source with more detail**
4. **Union source references**
5. **Take highest confidence** for ambiguous fields

## Query Patterns

### By Location + Date

```
GET /events?city=manchester&date=2025-06-15

GET /events?lat=53.48&lng=-2.24&radius=5km&date_from=today&date_to=+7days
```

### By Venue

```
GET /venues/{venueId}/events?status=confirmed
```

### By Artist

```
GET /artists/{artistId}/events?status=confirmed
```

### Tonight View

```
GET /events/tonight?city=london&genre=jazz
```

## Related

- [[04-architecture/data-model|Data Model]]
- [[venue-model]]
- [[artist-model]]
- [[03-backlog/now|BNDY-002: Event ingestion pipeline]]
