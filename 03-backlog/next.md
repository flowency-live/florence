# Next

Upcoming work - ready to start when current sprint completes.

---

## BNDY-004: Create canonical artist model

**Status:** Ready
**Priority:** P1
**Owner:** TBD

### Why

Artists are the second key entity. Linking events to artists enables:
- Artist pages
- Artist-based recommendations
- "Similar artists" discovery

### Outcome

Canonical artist identity that can be linked from events.

### Acceptance Criteria

- [ ] Artist has canonical ID
- [ ] Artist can have aliases (stage names, band name variants)
- [ ] Artist has genre associations
- [ ] Artist can be linked to events (performed at)
- [ ] Artist can have external links (Spotify, Bandcamp, etc.)

---

## BNDY-005: Add genre/vibe tagging

**Status:** Ready
**Priority:** P1
**Owner:** TBD

### Why

Users want to filter by genre. AI can infer genre from artist data.

### Outcome

Events and artists have genre tags that users can filter by.

### Acceptance Criteria

- [ ] Genre taxonomy defined
- [ ] AI can auto-tag events based on artist
- [ ] Users can filter feed by genre
- [ ] Community can suggest/correct tags

---

## BNDY-006: Venue claim flow

**Status:** Needs design
**Priority:** P1
**Owner:** TBD

### Why

Community builders need to claim their venues to manage events and analytics.

### Outcome

Venue owners/managers can claim and verify ownership of venue profiles.

### Acceptance Criteria

- [ ] User can request to claim a venue
- [ ] Verification flow (email domain, document, or manual review)
- [ ] Claimed venue shows "Verified" badge
- [ ] Claimant can edit venue details

---

## BNDY-007: Event submission form

**Status:** Needs design
**Priority:** P1
**Owner:** TBD

### Why

Not all events can be scraped. Community builders need to submit directly.

### Outcome

Verified venues/promoters can submit events to bndy.

### Acceptance Criteria

- [ ] Authenticated users can submit events
- [ ] Events linked to verified venue
- [ ] Events go live immediately (trusted submitters) or moderation queue
- [ ] Duplicate detection warns before submission

---

## Related

- [[now]] - Current sprint
- [[later]] - Future ideas
