# Migration Execution Plan

Real-world migration from v1 DynamoDB to canonical v2 model.

## Current State: v1 DynamoDB Tables

### Core Entity Tables

| Table | Items | PK | SK | Purpose |
|-------|-------|----|----|---------|
| `bndy-artists` | 275 | `id` | — | Artist/band profiles |
| `bndy-venues` | 284 | `id` | — | Venue locations |
| `bndy-events` | 8 | `id` | — | Events (low count = beta) |

### Supporting Tables

| Table | PK | SK | Purpose |
|-------|----|----|---------|
| `bndy-artist-memberships` | `membership_id` | — | Band membership (6 items) |
| `bndy-songs` | `id` | — | Song catalog |
| `bndy-artist-songs` | `artistSongId` | — | Song pipeline |
| `bndy-setlists` | `setlistId` | — | Setlist templates |
| `bndy-artist-venues` | `artistId+venueId` | — | CRM relationships |
| `bndy-artist-venue-contacts` | `contactId` | — | Venue contacts |
| `bndy-venue-notes` | `noteId` | — | Private notes |
| `bndy-users` | `userId` | — | User accounts |

### Auth/Ephemeral Tables

| Table | PK | TTL | Purpose |
|-------|----|----|---------|
| `bndy-invites` | `token` | 7d | Membership invites |
| `bndy-magic-tokens` | `token` | 5min | Email magic links |
| `bndy-otp-codes` | `phoneNumber` | 15min | Phone OTP |
| `bndy-oauth-states` | `state` | 10min | CSRF protection |
| `bndy-notifications` | `notificationId` | 30d | In-app notifications |

---

## Current PK/SK Patterns

### Events Table (`bndy-events`)

```
PK: id (UUID)
SK: — (no sort key)

GSIs:
- ownerUserId-date-index: PK=ownerUserId, SK=date
- artistId-date-index: PK=artistId, SK=date
- venueId-date-index: PK=venueId, SK=date
- geohash6-date-index: PK=geohash6, SK=date
```

**Problem:** Single artistId field, but events can have multiple artists. GSI only indexes primary artist.

### Venues Table (`bndy-venues`)

```
PK: id (UUID)
SK: — (no sort key)

No GSIs currently.
```

**Problem:** No geospatial index. Client-side filtering for location queries.

### Artists Table (`bndy-artists`)

```
PK: id (UUID)
SK: — (no sort key)

No GSIs currently.
```

---

## Current Geo Model

### Event Geospatial

```typescript
// Event has redundant geo fields
location: { lat: number; lng: number };  // Structured
geoLat?: number;                          // Legacy flat
geoLng?: number;                          // Legacy flat
geohash4?: string;                        // Rough (city-level)
geohash6?: string;                        // Precise (neighborhood)
```

**GSI:** `geohash6-date-index` enables queries like "events in this area after this date"

### Venue Geospatial

```typescript
location: { lat: number; lng: number };
latitude?: number;                        // Legacy
longitude?: number;                       // Legacy
googlePlaceId?: string;                   // External reference
```

**Problem:** No geohash on venues. Can't efficiently query "venues in area".

---

## Current Duplicate Logic (4-Level)

From `VenuesFunction`:

| Level | Confidence | Matching Criteria |
|-------|------------|-------------------|
| 1 | 100% | Google Place ID exact match |
| 2 | 90% | Within 50m radius + 80% name similarity |
| 3 | 70% | 85% name similarity + 50% address token overlap |
| 4 | 0% | No match → create new venue |

**Implementation:** `find-or-create` endpoint runs this on venue creation.

**Gap:** No equivalent for artists. No post-hoc deduplication workflow.

---

## Existing Indexes

### Implemented GSIs

| Table | Index | PK | SK | Use Case |
|-------|-------|----|----|----------|
| `bndy-events` | `ownerUserId-date` | ownerUserId | date | User's personal events |
| `bndy-events` | `artistId-date` | artistId | date | Artist's gig history |
| `bndy-events` | `venueId-date` | venueId | date | Venue's event history |
| `bndy-events` | `geohash6-date` | geohash6 | date | Location-based discovery |
| `bndy-artist-memberships` | `artist_id-index` | artist_id | — | Members of a band |
| `bndy-artist-memberships` | `user_id-index` | user_id | — | User's band memberships |
| `bndy-artist-songs` | `artistId-status-index` | artistId | status | Song pipeline by status |
| `bndy-notifications` | `userId-createdAt-index` | userId | createdAt | User's notifications |

### Missing Indexes (Migration Should Add)

| Table | Needed Index | Use Case |
|-------|--------------|----------|
| `bndy-venues` | `city-index` | Venues by city |
| `bndy-venues` | `geohash6-index` | Venues by location |
| `bndy-artists` | `genre-index` | Artists by genre |
| Future | `sourceId-index` | Entities by source |

---

## Inferred Relationships in Current Data

### Explicit Relationships

| Relationship | Storage | Count |
|--------------|---------|-------|
| Event → Venue | `event.venueId` | All events |
| Event → Artist | `event.artistId` / `event.artistIds[]` | All events |
| User → Artist | `bndy-artist-memberships` | 6 records |
| Artist → Venue (CRM) | `bndy-artist-venues` | Unknown |
| Artist → Song | `bndy-artist-songs` | Unknown |

### Derivable Relationships

From existing data, we can derive:

| Relationship | Derivation | Migration Action |
|--------------|------------|------------------|
| `artist_played_venue` | GROUP BY artistId, venueId FROM events | Create relationship records |
| `venue_hosts_genre` | Aggregate artist genres per venue | Compute on migration |
| `artist_active_in_area` | Events by artist grouped by geohash4 | Compute on migration |

### Membership Graph

The `bndy-artist-memberships` table already models:
- `musician_member_of_artist` (user → artist with role)
- Permissions per membership
- Invite tracking

**Migration:** Preserve this model, add relationship records.

---

## Migration Sequencing

### Phase 0: Preparation (No Data Changes)

```
1. Create v2 canonical table structure (new tables)
2. Deploy migration Lambda functions
3. Set up parallel-write infrastructure
4. Create migration audit dashboard
```

### Phase 1: Venues First

**Why first:** Venues are the anchor. Events and artists reference venues.

```
1. Read all bndy-venues records
2. For each venue:
   a. Create SourceRecord (sourceType: migration, sourceName: bndy_v1)
   b. Generate canonical ID (vnue_xxx)
   c. Run 4-level duplicate detection against already-migrated
   d. Calculate confidence score
   e. Write canonical venue with legacyIds.bndyV1 = original id
   f. Add geohash6 if missing
3. Build venueId → canonicalId mapping table
```

**Parallel-run:** Both tables active. Writes go to both.

### Phase 2: Artists Second

**Why second:** Artists are referenced by events.

```
1. Read all bndy-artists records
2. For each artist:
   a. Create SourceRecord
   b. Generate canonical ID (arts_xxx)
   c. Run name-similarity duplicate detection
   d. Calculate confidence score
   e. Write canonical artist with legacyIds.bndyV1
   f. Preserve membership links (update membership records)
3. Build artistId → canonicalId mapping table
```

### Phase 3: Events Third

**Why third:** Events reference both venues and artists.

```
1. Read all bndy-events records
2. For each event:
   a. Create SourceRecord
   b. Generate canonical ID (evnt_xxx)
   c. Resolve venueId → canonical venueId using mapping
   d. Resolve artistId(s) → canonical artistId(s) using mapping
   e. Calculate confidence score
   f. Add provenance fields
   g. Write canonical event
3. Build eventId → canonicalId mapping table
```

### Phase 4: Relationships

```
1. For each canonical event:
   a. Create artist_played_event relationship
   b. Create/update artist_played_venue relationship
   c. Increment occurrenceCount for repeat relationships
2. Compute derived relationships:
   a. artist_similar_to (shared venues + genres)
   b. venue_books_genre (aggregate from events)
```

### Phase 5: Memberships & CRM

```
1. Migrate bndy-artist-memberships → update with canonical artistIds
2. Migrate bndy-artist-venues → update with canonical IDs
3. Migrate bndy-artist-venue-contacts
4. Migrate bndy-venue-notes
```

### Phase 6: Cutover

```
1. Verify record counts match
2. Verify relationship integrity
3. Switch read paths to canonical tables
4. Keep v1 tables read-only for rollback (30 days)
5. Archive v1 tables to S3
```

---

## Parallel-Run Strategy

### During Migration (Weeks 1-2)

```
┌─────────────┐     ┌─────────────┐
│   v1 API    │     │   v2 API    │
└──────┬──────┘     └──────┬──────┘
       │                   │
       ▼                   ▼
┌─────────────┐     ┌─────────────┐
│ bndy-venues │     │ canonical-  │
│ bndy-artists│     │ venues      │
│ bndy-events │     │ canonical-  │
└─────────────┘     │ artists     │
                    │ canonical-  │
                    │ events      │
                    └─────────────┘
```

### Write Strategy

| Operation | v1 Table | v2 Table |
|-----------|----------|----------|
| Create | Write | Write (with canonical ID) |
| Update | Write | Write (update canonical) |
| Read | Primary | Shadow (for comparison) |
| Delete | Soft delete | Soft delete |

### Comparison Checks

Run daily:
```
- Record count: v1 vs v2
- Field coverage: % of fields migrated
- Relationship integrity: all refs resolve
- Query parity: same results from both
```

---

## Canonical Table Design

### Single-Table Design for Canonical Entities

```
PK                      SK                          Type
VENUE#vnue_abc123       METADATA                    Venue
VENUE#vnue_abc123       SOURCE#src_001              SourceRef
VENUE#vnue_abc123       ALIAS#kings-arms            Alias
ARTIST#arts_xyz789      METADATA                    Artist
ARTIST#arts_xyz789      SOURCE#src_002              SourceRef
ARTIST#arts_xyz789      REL#played_venue#vnue_abc   Relationship
EVENT#evnt_def456       METADATA                    Event
EVENT#evnt_def456       SOURCE#src_003              SourceRef
EVENT#evnt_def456       ARTIST#arts_xyz789          EventArtist link
```

### GSIs for Canonical Tables

| GSI | PK | SK | Use Case |
|-----|----|----|----------|
| GSI1 | `entityType` | `createdAt` | All venues/artists/events sorted |
| GSI2 | `city` | `entityType#name` | Entities by city |
| GSI3 | `geohash6` | `date` | Location + time queries |
| GSI4 | `legacyId` | — | Lookup by v1 ID |
| GSI5 | `status` | `confidenceScore` | Review queues |

---

## Data Transformation Rules

### Venue Transformation

```typescript
// v1 → v2
{
  // Identity
  id: venue.id,                           // Keep for legacyIds
  canonicalId: generateId('vnue'),        // New canonical

  // Naming
  name: venue.name,
  aliases: venue.nameVariants || [],
  slug: slugify(venue.name),

  // Location
  coordinates: venue.location || {
    lat: venue.latitude,
    lng: venue.longitude
  },
  geohash6: computeGeohash(coordinates, 6),
  address: {
    line1: venue.address,
    city: venue.city,
    postcode: venue.postcode,
    country: 'UK'
  },

  // Verification
  status: venue.validated ? 'verified' : 'unverified',
  confidenceScore: calculateVenueConfidence(venue),

  // Provenance
  sources: [{
    sourceId: generateId('src'),
    sourceType: 'migration',
    sourceName: 'bndy_v1'
  }],
  legacyIds: { bndyV1: venue.id },

  // Timestamps
  createdAt: venue.createdAt,
  updatedAt: new Date().toISOString(),
  migratedAt: new Date().toISOString()
}
```

### Artist Transformation

```typescript
// v1 → v2
{
  canonicalId: generateId('arts'),
  name: artist.name,
  aliases: artist.nameVariants || [],
  artistType: artist.artist_type || artist.artistType,
  genres: artist.genres || [],

  // Profile
  bio: artist.bio,
  location: artist.location,

  // Links (normalize)
  socialLinks: normalizeSocialLinks(artist),
  musicLinks: normalizeMusicLinks(artist),

  // Ownership
  status: artist.claimedByUserId ? 'claimed' :
          artist.isVerified ? 'verified' : 'unverified',
  claimedBy: artist.claimedByUserId,

  // Provenance
  confidenceScore: calculateArtistConfidence(artist),
  sources: [{ sourceType: 'migration', sourceName: 'bndy_v1' }],
  legacyIds: { bndyV1: artist.id }
}
```

### Event Transformation

```typescript
// v1 → v2
{
  canonicalId: generateId('evnt'),
  name: event.name || event.title,

  // When
  startDate: event.date,
  startTime: event.startTime,
  endTime: event.endTime,
  timezone: 'Europe/London',

  // Where (resolve to canonical)
  venueId: venueMapping[event.venueId],
  venue: {
    id: venueMapping[event.venueId],
    name: event.venueName,
    city: event.venueCity
  },

  // Who (resolve to canonical)
  artists: resolveArtists(event, artistMapping),

  // What
  eventType: event.type || 'gig',
  description: event.description,
  genres: inferGenres(event),

  // Provenance
  status: mapEventStatus(event.status),
  confidenceScore: calculateEventConfidence(event),
  sources: [{ sourceType: 'migration', sourceName: 'bndy_v1' }],
  provenance: {
    verificationStatus: 'unverified',
    uncertaintyFlags: detectUncertainty(event)
  },

  legacyIds: { bndyV1: event.id }
}
```

---

## Confidence Score Calculation

### Venue Confidence

```typescript
function calculateVenueConfidence(venue: V1Venue): number {
  let score = 0.5;

  if (venue.location?.lat && venue.location?.lng) score += 0.15;
  if (venue.postcode) score += 0.1;
  if (venue.googlePlaceId) score += 0.1;
  if (venue.website) score += 0.05;
  if (venue.validated) score += 0.1;

  // Activity bonus
  const eventCount = await countVenueEvents(venue.id);
  if (eventCount > 10) score += 0.1;
  else if (eventCount > 0) score += 0.05;

  return Math.min(score, 1);
}
```

### Artist Confidence

```typescript
function calculateArtistConfidence(artist: V1Artist): number {
  let score = 0.5;

  if (artist.bio) score += 0.05;
  if (artist.genres?.length > 0) score += 0.05;
  if (artist.spotifyUrl || artist.bandcampUrl) score += 0.1;
  if (artist.claimedByUserId) score += 0.15;
  if (artist.isVerified) score += 0.1;

  // Activity bonus
  const eventCount = await countArtistEvents(artist.id);
  if (eventCount > 5) score += 0.1;
  else if (eventCount > 0) score += 0.05;

  return Math.min(score, 1);
}
```

---

## Rollback Plan

### Trigger Conditions

- Record count mismatch > 1%
- Query failures > 5%
- User-reported data loss
- Relationship integrity failures

### Rollback Steps

```
1. Switch read paths back to v1 tables (immediate)
2. Disable writes to v2 tables
3. Investigate root cause
4. Fix migration logic
5. Re-run migration from snapshot
```

### Data Preservation

- v1 tables remain untouched during migration
- v1 tables set to read-only after cutover
- v1 tables archived to S3 after 30 days
- S3 archive retained for 1 year

---

## Success Criteria

| Metric | Target |
|--------|--------|
| Venues migrated | 284/284 (100%) |
| Artists migrated | 275/275 (100%) |
| Events migrated | 8/8 (100%) |
| Duplicate pairs resolved | All detected |
| Relationships created | ≥ event count |
| Confidence scores assigned | 100% |
| Legacy IDs preserved | 100% |
| Query parity | 100% same results |
| Downtime | 0 (parallel run) |

---

## Timeline

| Week | Phase | Deliverable |
|------|-------|-------------|
| 1 | Preparation | v2 tables created, Lambda deployed |
| 1-2 | Venues | 284 venues migrated |
| 2 | Artists | 275 artists migrated |
| 2-3 | Events | 8 events migrated |
| 3 | Relationships | Graph populated |
| 3-4 | Parallel run | Comparison passing |
| 4 | Cutover | v2 primary, v1 read-only |

---

## Related

- [[v1-to-canonical-plan]] - Strategic context
- [[canonical-id-strategy]] - ID generation
- [[v1-source-record-strategy]] - Source records
- [[../08-v1-platform/data-models|v1 Data Models]] - Current schemas
- [[../05-entities/source-record-model|Source Record Model]] - v2 provenance
- [[../05-entities/relationship-model|Relationship Model]] - v2 graph
