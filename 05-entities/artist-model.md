# Artist Model

Performers and acts in the bndy ecosystem.

## Why Artists Matter

Artists are the creative core:
- Users follow artists to discover events
- Genre signals come from artist profiles
- Artist draw affects venue booking decisions

## Core Challenges

### Identity Resolution

Same artist, many names:
- Stage names vs legal names
- Band name variations ("The National" vs "National")
- Solo artists in multiple projects
- DJ aliases

### Attribution

- Who's actually performing at an event?
- Headline vs support acts
- Collaborations and one-off projects

## Schema

```typescript
interface Artist {
  // Identity
  id: string;                      // Canonical UUID
  name: string;                    // Primary display name
  aliases: string[];               // Stage names, variations
  slug: string;                    // URL-safe identifier

  // Classification
  artistType: ArtistType;
  genres: string[];                // ["indie rock", "post-punk"]
  subgenres?: string[];            // More specific tags

  // Profile
  bio?: string;
  hometown?: string;               // "Manchester, UK"
  formedYear?: number;
  memberCount?: number;            // For bands

  // Links
  website?: string;
  socialLinks?: {
    instagram?: string;
    facebook?: string;
    twitter?: string;
    tiktok?: string;
  };
  musicLinks?: {
    spotify?: string;
    appleMusic?: string;
    bandcamp?: string;
    soundcloud?: string;
    youtube?: string;
  };

  // Media
  imageUrl?: string;
  images?: string[];

  // Verification
  status: ArtistStatus;
  confidenceScore: number;
  claimedBy?: string;
  claimedAt?: string;

  // Source tracking
  sources: SourceReference[];

  // Activity
  gigCount?: number;               // Total events played
  venueCount?: number;             // Unique venues
  cityCount?: number;              // Cities performed in
  lastGigDate?: string;

  // Timestamps
  createdAt: string;
  updatedAt: string;
}

type ArtistType =
  | 'solo'
  | 'band'
  | 'duo'
  | 'dj'
  | 'collective'
  | 'orchestra'
  | 'choir'
  | 'other';

type ArtistStatus =
  | 'active'
  | 'unverified'
  | 'verified'
  | 'claimed'
  | 'inactive'
  | 'disbanded';
```

## Resolution Rules

### Matching Criteria

Two artist records likely refer to the same artist if:

| Signal | Weight | Notes |
|--------|--------|-------|
| Same Spotify/Bandcamp ID | Very High | Definitive |
| High name similarity + same genres | High | Common case |
| Same social handles | High | Reliable |
| Played same events | Medium | Correlation |

### Genre Inference

If artist has no explicit genres:

1. Check Spotify genres via API
2. Analyse event genres they're booked on
3. Check venue types they play
4. LLM inference from bio/description

### Merge Strategy

1. **Keep canonical ID** from higher-confidence record
2. **Merge aliases** from both
3. **Union music links** (more discoverability)
4. **Prefer claimed** data
5. **Sum activity metrics** carefully (avoid double-counting)

## Relationships

### Artist ↔ Event

```typescript
interface EventArtistLink {
  eventId: string;
  artistId: string;
  role: 'headliner' | 'support' | 'resident' | 'special_guest';
  billingOrder?: number;           // 1 = top of bill
}
```

### Artist ↔ Venue

Derived relationship:
- Artists who frequently play a venue
- Useful for venue recommendations
- Calculated, not stored

## API Examples

### Get Artist

```
GET /artists/{id}

Response:
{
  "id": "arts_xyz789",
  "name": "Dry Cleaning",
  "artistType": "band",
  "genres": ["post-punk", "art rock"],
  "hometown": "London, UK",
  "upcomingGigCount": 5,
  "musicLinks": {
    "spotify": "https://open.spotify.com/artist/...",
    "bandcamp": "https://drycleaning.bandcamp.com"
  }
}
```

### Search Artists

```
GET /artists?genre=jazz&city=bristol&limit=20
```

## Related

- [[04-architecture/data-model|Data Model]]
- [[venue-model]]
- [[event-model]]
