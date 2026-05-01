# Now

Current sprint - actively being worked on.

---

## BNDY-001: Create canonical venue model

**Status:** Ready
**Priority:** P0
**Owner:** TBD

### Why

Venues are the anchor entity for grassroots discovery. Without canonical venue identity, we can't:
- De-duplicate events
- Build venue pages
- Track venue programming over time

### Outcome

One canonical venue identity across scraped, submitted, and verified sources.

### Acceptance Criteria

- [ ] Venue has canonical ID (UUID)
- [ ] Venue can have aliases (alternative names)
- [ ] Venue can have source references (where we learned about it)
- [ ] Venue can have confidence score (data quality indicator)
- [ ] Venue can be community verified (human-validated)
- [ ] Venue has location (lat/lng, address, city)
- [ ] Venue has capacity (if known)
- [ ] Venue has genre associations

### Notes

See [[05-entities/venue-model]] for full data model.

---

## BNDY-002: Set up event ingestion pipeline

**Status:** Ready
**Priority:** P0
**Owner:** TBD

### Why

Discovery requires data. We need to ingest events from external sources before users can find them.

### Outcome

Automated pipeline that ingests events from at least 3 initial sources.

### Acceptance Criteria

- [ ] Pipeline runs on schedule (daily minimum)
- [ ] Events are normalised to canonical schema
- [ ] Events are linked to canonical venue (or flagged for resolution)
- [ ] Duplicate events are detected and merged
- [ ] Failed ingestions are logged for review

### Notes

See [[04-architecture/ingestion-pipeline]] for architecture.

---

## BNDY-003: Build basic event feed UI

**Status:** Blocked (needs BNDY-001, BNDY-002)
**Priority:** P0
**Owner:** TBD

### Why

Users need to see events. The feed is the core product surface.

### Outcome

A web-based feed showing upcoming events in a location.

### Acceptance Criteria

- [ ] User can view events by city
- [ ] Events show: name, venue, date, time, genre tags
- [ ] Events link to external ticket/info URL
- [ ] Feed is paginated or infinite scroll
- [ ] Feed is mobile-responsive

---

## Related

- [[next]] - What comes after
- [[later]] - Future ideas
- [[risks]] - Known risks
