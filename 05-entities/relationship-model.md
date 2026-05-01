# Relationship Model

The graph edges that make bndy's intelligence defensible.

## Why Relationships Matter

The long-term defensible asset is not the event table. It is the **relationship graph**.

Raw event data is commoditised. Understanding how entities connect - that's the moat.

## Schema

```typescript
interface Relationship {
  // Identity
  relationshipId: string;              // UUID

  // Endpoints
  fromEntityType: EntityType;
  fromEntityId: string;
  relationshipType: RelationshipType;
  toEntityType: EntityType;
  toEntityId: string;

  // Strength
  confidence: number;                  // 0-1 how sure are we
  strength: number;                    // 0-1 how strong is the relationship

  // Evidence
  sourceIds: string[];                 // SourceRecords that prove this
  eventIds?: string[];                 // Events that demonstrate this

  // Temporality
  firstSeen: string;                   // When we first observed this
  lastSeen: string;                    // Most recent observation
  occurrenceCount: number;             // How many times observed

  // Metadata
  metadata?: Record<string, any>;      // Relationship-specific data

  // Timestamps
  createdAt: string;
  updatedAt: string;
}

type EntityType =
  | 'artist'
  | 'musician'
  | 'venue'
  | 'event'
  | 'promoter'
  | 'genre'
  | 'scene'
  | 'city'
  | 'area'
  | 'recurring_night'
  | 'festival';

type RelationshipType =
  // Artist relationships
  | 'artist_played_event'
  | 'artist_played_venue'
  | 'artist_similar_to'
  | 'artist_shares_members_with'
  | 'artist_has_genre'
  | 'artist_from_area'
  | 'artist_has_draw_in_area'

  // Musician relationships
  | 'musician_member_of'
  | 'musician_available_as_dep'
  | 'musician_plays_instrument'

  // Venue relationships
  | 'venue_hosted_event'
  | 'venue_books_genre'
  | 'venue_hosts_recurring_night'
  | 'venue_in_area'
  | 'venue_similar_to'

  // Promoter relationships
  | 'promoter_promoted_event'
  | 'promoter_books_artist'
  | 'promoter_works_with_venue'
  | 'promoter_operates_in_area'

  // Event relationships
  | 'event_part_of_series'
  | 'event_part_of_festival'

  // Scene relationships
  | 'scene_includes_venue'
  | 'scene_includes_artist'
  | 'scene_in_area';
```

## Key Relationships

### Artist ↔ Venue History

```typescript
// Derived from events
{
  relationshipType: 'artist_played_venue',
  fromEntityId: 'artist_123',
  toEntityId: 'venue_456',
  occurrenceCount: 7,           // Played here 7 times
  firstSeen: '2023-01-15',
  lastSeen: '2025-04-20',
  strength: 0.85,               // Strong relationship
  metadata: {
    avgCrowdSize: 'good',
    venueRating: 4.5
  }
}
```

### Artist Similarity

```typescript
// Computed from shared venues, genres, audiences
{
  relationshipType: 'artist_similar_to',
  fromEntityId: 'artist_123',
  toEntityId: 'artist_789',
  confidence: 0.78,
  metadata: {
    sharedVenues: 5,
    genreOverlap: 0.8,
    audienceOverlap: 0.6,
    similarityReason: 'Same venues, similar genre'
  }
}
```

### Shared Members

```typescript
// When musicians play in multiple bands
{
  relationshipType: 'artist_shares_members_with',
  fromEntityId: 'artist_123',
  toEntityId: 'artist_456',
  metadata: {
    sharedMembers: ['musician_001', 'musician_002']
  }
}
```

### Venue Genre Affinity

```typescript
// What genres does this venue book?
{
  relationshipType: 'venue_books_genre',
  fromEntityId: 'venue_456',
  toEntityId: 'genre_jazz',
  strength: 0.9,
  occurrenceCount: 45,          // 45 jazz events
  metadata: {
    percentageOfBookings: 0.6
  }
}
```

### Promoter Networks

```typescript
// Who works with whom?
{
  relationshipType: 'promoter_works_with_venue',
  fromEntityId: 'promoter_789',
  toEntityId: 'venue_456',
  occurrenceCount: 12,
  strength: 0.8,
  metadata: {
    avgEventSize: 'medium',
    relationship: 'regular'
  }
}
```

## Graph Queries

These relationships enable powerful queries:

| Query | How |
|-------|-----|
| "Venues that book jazz in Manchester" | venue_books_genre + venue_in_area |
| "Artists similar to Band X" | artist_similar_to |
| "Where could my band get booked?" | Find venues that book similar artists |
| "Who promotes indie rock in Leeds?" | promoter_books_artist (genre) + promoter_operates_in_area |
| "Find me a dep for drums" | musician_available_as_dep + musician_plays_instrument |

## Building Relationships

### Explicit (from events)

Every event creates relationships:
- `artist_played_event`
- `event_hosted_at_venue`
- `artist_played_venue` (derived)

### Inferred (from patterns)

AI can infer:
- `artist_similar_to` (shared venues + genres)
- `venue_similar_to` (similar bookings)
- `artist_has_draw_in_area` (repeated local bookings)

### Contributed (from community)

Users can add:
- `musician_member_of` (band membership)
- `musician_available_as_dep` (dep network)
- Scene curation

## Storage in DynamoDB

Graph overlay pattern:

```
PK                    SK                           Data
ENTITY#artist_123     METADATA                     Artist record
ENTITY#artist_123     REL#played_venue#venue_456   Relationship
ENTITY#artist_123     REL#similar_to#artist_789    Relationship
ENTITY#venue_456      REL#books_genre#jazz         Relationship
```

GSI for reverse lookups:
- Find all artists that played at venue X
- Find all venues that book genre Y

## Related

- [[artist-model]]
- [[venue-model]]
- [[01-strategy/ai-native-reframe|AI-Native Reframe]]
