# bndy v1 Technical Debt

Known issues, gaps, and improvement opportunities in the v1 platform.

## Critical (P0)

### No Lambda Input Validation

**Impact:** Type confusion risks, potential security vulnerabilities

All handlers use raw `JSON.parse(event.body)` without Zod or any runtime validation:

```javascript
// Current (dangerous)
const body = JSON.parse(event.body);
const { name, venueId } = body;  // No validation

// Required
const body = EventInputSchema.parse(JSON.parse(event.body));
```

Unvalidated fields are stored directly in DynamoDB.

**Fix:** Add Zod schemas to all Lambda handlers.

---

## High Priority (P1)

### Type Safety Violations

| Issue | Count | Location |
|-------|-------|----------|
| Explicit `: any` types | 180 | 71 files |
| Type assertions without validation | 321 | Throughout |

### Service Layer Bypass

- 46 backstage files make 204 direct API calls
- Should go through service layer for consistency
- Makes refactoring difficult

### Event Conflict Checking Stubbed

- Firebase integration removed
- DynamoDB endpoint not implemented
- Users can double-book without warning

---

## Missing Functionality

### Frontstage Gaps

- No event update/delete endpoints
- No venue edit endpoint
- No admin approval UI

### Backend Gaps

- Venue admin functions not implemented
- No bulk operations
- No data export

---

## Code Quality Issues

### Inconsistent Naming

| Pattern | Examples |
|---------|----------|
| camelCase vs snake_case | `createdAt` vs `created_at` |
| Duplicate fields | `artist_type` vs `artistType` |
| Legacy aliases | `geoLat/geoLng` vs `location.lat/lng` |

### Test Coverage

- Current: ~18%
- Target: 80%+
- Critical paths untested

### Error Handling

Many handlers swallow errors or return generic 500s:

```javascript
// Current
catch (error) {
  console.error(error);
  return { statusCode: 500, body: 'Internal error' };
}

// Required
catch (error) {
  if (error instanceof ValidationError) {
    return { statusCode: 400, body: error.message };
  }
  throw error;  // Let it surface
}
```

---

## Architecture Issues

### Single Lambda Layer

- `bndy-jwt:2` bundles aws-sdk v2 + jsonwebtoken
- aws-sdk v3 is current
- Layer size: 14.5MB (bloated)

### No CDK/IaC

- Infrastructure created via console
- No reproducible deployments
- Environment drift possible

### No CI/CD

- Manual deployments
- No automated testing
- No staging environment validation

---

## Data Quality Issues

### Duplicate Entities

Despite 4-level venue deduplication, duplicates exist:
- Venues with slight name variations
- Artists with multiple profiles
- Events from multiple sources

### Orphaned Records

- Events referencing deleted venues
- Memberships for deleted artists
- No referential integrity checks

### Legacy Fields

Many fields exist for backwards compatibility:
- `geoLat/geoLng` (use `location.lat/lng`)
- `artistId` (use `artistIds[]`)
- `facebookUrl` (use `socialMediaUrls[]`)

---

## Performance Issues

### Client-Side Distance Filtering

```typescript
// Current: fetch all events, filter client-side
const events = await fetchAllPublicEvents();
const nearby = events.filter(e => haversine(e, userLocation) < 10);

// Better: geohash query server-side
const events = await fetchEvents({ geohash6: userGeohash });
```

### No Pagination

Some endpoints return unbounded results:
- `GET /api/artists` - all 275 artists
- `GET /api/venues` - all 284 venues

---

## Debt Prioritization for v2

### Must Fix (Blocks v2)

1. Lambda input validation (security)
2. Type safety (DX, reliability)
3. CDK infrastructure (reproducibility)

### Should Fix (Improves v2)

1. Service layer consistency
2. Test coverage
3. CI/CD pipeline

### Can Defer

1. Legacy field cleanup (migration handles)
2. AWS SDK v3 upgrade (not blocking)
3. Performance optimizations (scale later)

## Related

- [[overview]]
- [[migration-notes]]
