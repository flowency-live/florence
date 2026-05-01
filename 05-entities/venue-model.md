# Venue Model

The anchor entity for grassroots music discovery.

## Why Venues Matter

Venues are the stable reference point in a chaotic data landscape:
- Events come and go, but venues persist
- Artists play many venues, but venue identity is clear
- Users discover by location → venue is the geographic anchor

## Core Challenges

### Identity Resolution

The same venue appears differently across sources:
- "The Blue Note" vs "Blue Note Jazz Club" vs "Bluenote"
- Different addresses (street vs postcode)
- Temporary closures, name changes, rebrandings

### Data Quality

Venue data is often:
- Incomplete (no capacity, no contact info)
- Incorrect (wrong coordinates, closed venues)
- Duplicated (same venue, multiple records)

## Schema

```typescript
interface Venue {
  // Identity
  id: string;                      // Canonical UUID (bndy-generated)
  name: string;                    // Primary display name
  aliases: string[];               // ["Blue Note", "Bluenote Jazz Club"]
  slug: string;                    // URL-safe: "blue-note-jazz-club"

  // Location
  address: {
    line1: string;                 // "131 West 3rd Street"
    line2?: string;
    city: string;                  // "New York"
    region?: string;               // "NY"
    postcode: string;              // "10012"
    country: string;               // "US"
  };
  coordinates: {
    lat: number;                   // 40.7308
    lng: number;                   // -74.0006
  };

  // Metadata
  capacity?: number;               // Max attendees
  venueType: VenueType;
  genres: string[];                // ["jazz", "blues"]
  description?: string;

  // Contact
  website?: string;
  email?: string;
  phone?: string;
  socialLinks?: {
    instagram?: string;
    facebook?: string;
    twitter?: string;
  };

  // Media
  imageUrl?: string;               // Primary venue photo
  images?: string[];               // Gallery

  // Verification
  status: VenueStatus;
  confidenceScore: number;         // 0-1 data quality
  verifiedAt?: string;             // When human-verified
  claimedBy?: string;              // User ID if claimed
  claimedAt?: string;

  // Source tracking
  sources: SourceReference[];

  // Activity
  eventCount?: number;             // Total events hosted
  lastEventDate?: string;          // Most recent event

  // Timestamps
  createdAt: string;
  updatedAt: string;
}

type VenueType =
  | 'bar'
  | 'pub'
  | 'club'
  | 'theatre'
  | 'concert_hall'
  | 'cafe'
  | 'restaurant'
  | 'outdoor'
  | 'warehouse'
  | 'community_space'
  | 'church'
  | 'other';

type VenueStatus =
  | 'active'           // Open, hosting events
  | 'unverified'       // Data exists, not validated
  | 'verified'         // Human-validated
  | 'claimed'          // Claimed by owner
  | 'closed'           // Permanently closed
  | 'temporarily_closed';
```

## Resolution Rules

### Matching Criteria

Two venue records likely refer to the same venue if:

| Signal | Weight | Notes |
|--------|--------|-------|
| Same coordinates (< 50m) | High | Most reliable |
| Same postcode + similar name | High | Good fallback |
| Same address line1 | Medium | Addresses vary |
| High name similarity (> 0.85) | Medium | Fuzzy matching |
| Same source ID | High | Re-import |

### Confidence Scoring

```typescript
function calculateConfidence(venue: Venue): number {
  let score = 0.5; // Base

  // Positive signals
  if (venue.coordinates) score += 0.15;
  if (venue.address.postcode) score += 0.1;
  if (venue.website) score += 0.05;
  if (venue.sources.length > 1) score += 0.1;

  // Verification bonus
  if (venue.status === 'verified') score += 0.1;
  if (venue.status === 'claimed') score += 0.15;

  return Math.min(score, 1);
}
```

### Merge Strategy

When merging duplicate venues:

1. **Keep canonical ID** from higher-confidence record
2. **Merge aliases** from both records
3. **Prefer claimed/verified** data over scraped
4. **Union source references**
5. **Take most recent** for mutable fields (website, etc.)

## API Examples

### Get Venue

```
GET /venues/{id}

Response:
{
  "id": "vnue_abc123",
  "name": "Blue Note Jazz Club",
  "city": "New York",
  "genres": ["jazz", "blues"],
  "upcomingEventCount": 12,
  "confidenceScore": 0.95,
  "status": "claimed"
}
```

### Search Venues

```
GET /venues?city=london&genre=jazz&limit=20

Response:
{
  "venues": [...],
  "total": 47,
  "nextCursor": "..."
}
```

## Related

- [[04-architecture/data-model|Data Model]]
- [[event-model]]
- [[03-backlog/now|BNDY-001: Create canonical venue model]]
