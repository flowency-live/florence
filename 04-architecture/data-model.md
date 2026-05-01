# Data Model

bndy entity-relationship model.

## Core Entities

```
┌─────────────┐       ┌─────────────┐       ┌─────────────┐
│    Venue    │◄──────│    Event    │──────►│   Artist    │
└─────────────┘       └─────────────┘       └─────────────┘
       │                     │                     │
       │                     │                     │
       ▼                     ▼                     ▼
┌─────────────┐       ┌─────────────┐       ┌─────────────┐
│   Source    │       │    User     │       │   Genre     │
│  Reference  │       │  Bookmark   │       │    Tag      │
└─────────────┘       └─────────────┘       └─────────────┘
```

## Entity Schemas

### Venue

The anchor entity - where music happens.

```typescript
interface Venue {
  id: string;                    // Canonical UUID
  name: string;                  // Primary display name
  aliases: string[];             // Alternative names

  // Location
  address: string;
  city: string;
  postcode: string;
  country: string;
  coordinates: {
    lat: number;
    lng: number;
  };

  // Metadata
  capacity?: number;
  venueType: 'bar' | 'club' | 'theatre' | 'outdoor' | 'other';
  genres: string[];              // Associated genres

  // Verification
  status: 'unverified' | 'verified' | 'claimed';
  confidenceScore: number;       // 0-1 data quality
  claimedBy?: string;            // User ID if claimed

  // Source tracking
  sources: SourceReference[];

  // Timestamps
  createdAt: string;
  updatedAt: string;
}
```

### Event

A specific occurrence at a venue.

```typescript
interface Event {
  id: string;                    // Canonical UUID
  name: string;                  // Event title

  // When
  startDate: string;             // ISO date
  startTime?: string;            // HH:mm
  endDate?: string;
  endTime?: string;

  // Where
  venueId: string;               // Canonical venue reference

  // Who
  artistIds: string[];           // Performing artists

  // What
  description?: string;
  genres: string[];
  eventType: 'gig' | 'festival' | 'open_mic' | 'dj_set' | 'other';

  // Links
  ticketUrl?: string;
  infoUrl?: string;
  imageUrl?: string;

  // Pricing
  price?: {
    currency: string;
    amount: number;
    isFree: boolean;
  };

  // Verification
  status: 'active' | 'cancelled' | 'postponed' | 'past';
  confidenceScore: number;

  // Source tracking
  sources: SourceReference[];

  // Timestamps
  createdAt: string;
  updatedAt: string;
}
```

### Artist

A performer or act.

```typescript
interface Artist {
  id: string;                    // Canonical UUID
  name: string;                  // Primary display name
  aliases: string[];             // Stage names, band name variants

  // Metadata
  genres: string[];
  artistType: 'solo' | 'band' | 'dj' | 'collective' | 'other';

  // Links
  spotifyUrl?: string;
  bandcampUrl?: string;
  soundcloudUrl?: string;
  websiteUrl?: string;

  // Verification
  status: 'unverified' | 'verified' | 'claimed';
  confidenceScore: number;
  claimedBy?: string;

  // Source tracking
  sources: SourceReference[];

  // Timestamps
  createdAt: string;
  updatedAt: string;
}
```

### Source Reference

Tracks where data came from.

```typescript
interface SourceReference {
  sourceType: 'scrape' | 'submission' | 'api' | 'manual';
  sourceName: string;            // e.g., 'facebook', 'dice', 'venue_website'
  sourceId?: string;             // ID in source system
  sourceUrl?: string;

  importedAt: string;
  rawData?: object;              // Original payload
}
```

### User

A bndy account.

```typescript
interface User {
  id: string;
  email: string;
  name?: string;

  // Role
  role: 'fan' | 'community_builder' | 'artist' | 'admin';

  // Claims
  claimedVenues: string[];       // Venue IDs
  claimedArtists: string[];      // Artist IDs

  // Preferences
  homeCity?: string;
  genres?: string[];

  // Timestamps
  createdAt: string;
  lastActiveAt: string;
}
```

## DynamoDB Single-Table Design

Using single-table design for efficiency.

| PK | SK | Entity |
|----|-----|--------|
| `VENUE#<id>` | `METADATA` | Venue |
| `VENUE#<id>` | `SOURCE#<sourceId>` | Source ref |
| `EVENT#<id>` | `METADATA` | Event |
| `EVENT#<id>` | `VENUE#<venueId>` | Event-Venue link |
| `EVENT#<id>` | `ARTIST#<artistId>` | Event-Artist link |
| `ARTIST#<id>` | `METADATA` | Artist |
| `USER#<id>` | `METADATA` | User |
| `CITY#<city>` | `EVENT#<date>#<id>` | City events GSI |

## Related

- [[05-entities/venue-model|Venue Model Details]]
- [[05-entities/artist-model|Artist Model Details]]
- [[05-entities/event-model|Event Model Details]]
