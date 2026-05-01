# bndy v1 → v2 Migration Notes

What to keep, change, and discard as we build the AI-native platform.

## Migration Philosophy

v2 is not a rewrite - it's a **strategic evolution**:

1. **Keep the data** - 275 artists, 284 venues are valuable
2. **Keep working patterns** - Multi-band membership, venue dedup
3. **Fix the gaps** - Validation, types, testing
4. **Add AI-native capabilities** - Ingestion, entity resolution, recommendations

---

## What to Keep

### Data Assets

| Asset | Count | Action |
|-------|-------|--------|
| Artists | 275 | Migrate, enhance with canonical IDs |
| Venues | 284 | Migrate, add confidence scores |
| Events | 8 | Migrate historical, focus on ingestion |
| Users | — | Migrate, preserve auth |

### Working Patterns

1. **Multi-band membership model** - Works well, users understand it
2. **4-level venue deduplication** - Solid algorithm, extend don't replace
3. **Song pipeline (suggestion → voting → ready)** - Good UX
4. **RAG status per member** - Unique feature, keep
5. **JWT httpOnly cookies** - Secure session management

### Domain Knowledge

- Artist types (band, solo, duo, collective)
- Event types (gig, rehearsal, unavailable)
- Membership roles and permissions
- UK-specific (postcodes, venues, scenes)

---

## What to Change

### Architecture Changes

| v1 | v2 | Reason |
|----|----|----|
| Console-created infra | CDK/IaC | Reproducibility |
| Raw JSON.parse | Zod validation | Type safety |
| 19 separate tables | Single-table design | Efficiency |
| Client-side filtering | Server-side geohash | Performance |
| Service layer bypass | Strict service layer | Maintainability |

### Data Model Changes

| v1 Field | v2 Field | Change |
|----------|----------|--------|
| `artist_type` | `artistType` | Consistent naming |
| `geoLat/geoLng` | `coordinates.lat/lng` | Structured |
| `artistId` (single) | `artistIds[]` | Always array |
| `facebookUrl` etc | `socialLinks.facebook` | Structured |
| (none) | `confidenceScore` | Add data quality |
| (none) | `sources[]` | Track provenance |

### Entity Model Changes

| v1 | v2 | Change |
|----|----|----|
| Basic venue | Canonical venue with resolution | Add entity identity |
| Basic artist | Canonical artist with aliases | Add matching |
| Event with venueId | Event with resolved venue | Add confidence |
| (none) | Community Builder entity | New persona |

---

## What to Discard

### Legacy Fields

Remove on migration:
- `geoLat/geoLng` (superseded by `location`)
- `artistId` single field (use `artistIds[]`)
- Individual social URL fields (use `socialMediaUrls[]`)
- `title` field on events (use `name` only)

### Deprecated Patterns

- `bndy-user-bands` table (migrated to memberships)
- Direct API calls bypassing services
- Firebase conflict checking (never completed)
- Centrestage admin dashboard (rebuild in v2)

### Technical Debt (Don't Migrate)

- 180 `: any` types
- 321 unvalidated assertions
- Inconsistent error handling

---

## Migration Strategy

### Phase 1: Foundation

1. Set up CDK infrastructure
2. Define v2 Zod schemas
3. Create migration scripts for data transform
4. Establish test coverage baseline

### Phase 2: Data Migration

1. Export v1 data to JSON
2. Transform to v2 schemas
3. Add canonical IDs and confidence scores
4. Load into v2 DynamoDB

### Phase 3: API Migration

1. Build v2 Lambda handlers with validation
2. Parallel run v1 and v2
3. Migrate traffic gradually
4. Decommission v1 endpoints

### Phase 4: Frontend Migration

1. Update service layer to v2 endpoints
2. Add new AI-native features
3. Deprecate legacy UX patterns

---

## Data Transformation Examples

### Venue Migration

```typescript
// v1
{
  id: "abc-123",
  name: "Blue Note Jazz Club",
  latitude: 51.5,
  longitude: -0.1,
  validated: true
}

// v2
{
  id: "vnue_abc123",           // Prefixed canonical ID
  name: "Blue Note Jazz Club",
  aliases: [],
  coordinates: { lat: 51.5, lng: -0.1 },
  status: "verified",          // From validated: true
  confidenceScore: 0.9,        // Computed
  sources: [{
    sourceType: "migration",
    sourceName: "bndy-v1",
    importedAt: "2026-05-01T00:00:00Z"
  }]
}
```

### Event Migration

```typescript
// v1
{
  id: "evt-456",
  artistId: "art-789",          // Single artist
  venueId: "abc-123",
  geoLat: 51.5,
  geoLng: -0.1
}

// v2
{
  id: "evnt_456",
  artistIds: ["arts_789"],      // Always array
  venueId: "vnue_abc123",       // Prefixed
  coordinates: { lat: 51.5, lng: -0.1 },
  sources: [{
    sourceType: "migration",
    sourceName: "bndy-v1"
  }]
}
```

---

## Risk Mitigation

### Data Loss Prevention

- Full backup before migration
- Reversible transforms
- Parallel run period

### User Disruption

- Maintain session cookies
- Preserve user IDs
- Gradual feature rollout

### Rollback Plan

- Keep v1 running during migration
- Feature flags for v2 functionality
- Database snapshots at each phase

---

## Success Criteria

| Metric | Target |
|--------|--------|
| Data migrated | 100% of artists, venues, users |
| Validation coverage | 100% of Lambda handlers |
| Type safety | 0 `any` types |
| Test coverage | 80%+ |
| Downtime | < 1 hour |

## Related

- [[technical-debt]]
- [[overview]]
- [[04-architecture/data-model|v2 Data Model]]
- [[05-entities/venue-model|v2 Venue Model]]
