# Next

Upcoming work organised by strategic phase.

---

## Phase 2: AI-Assisted Event Intake

Allow users to submit signals, not structured data. AI extracts, humans confirm.

---

### BNDY-007: Poster/image upload

**Status:** ✅ DONE (in BUILD-001)
**Priority:** P1
**Phase:** 2

### Why

Users have posters, screenshots, flyers. Let them submit these instead of typing.

### Outcome

Users can upload an image and bndy extracts event data.

### Acceptance Criteria

- [x] User can upload image (poster, flyer, screenshot) via /dropzone
- [x] Image stored in S3 as base64
- [x] Signal created with signalType: 'image'
- [x] Triggers Step Functions workflow

---

### BNDY-008: AI extraction into candidate format

**Status:** ✅ DONE (in BUILD-002)
**Priority:** P1
**Phase:** 2

### Why

This is the shift from CRUD to signal processing. AI does the heavy lifting.

### Outcome

AI extracts structured claims from text/images.

### Acceptance Criteria

- [x] OCR extracts text from images (Textract)
- [x] LLM generates claims: event, artist, venue, date, time
- [x] Strength scores (weak/moderate/strong) with reasoning
- [x] Uncertainties explicitly flagged
- [x] Claims stored in DynamoDB for review

---

### BNDY-009: Human confirmation before publishing

**Status:** Ready
**Priority:** P1
**Phase:** 2

### Why

AI extracts, humans verify. This is how quality scales.

### Outcome

Extracted events go through human confirmation before publishing.

### Acceptance Criteria

- [ ] Candidate events appear in review queue
- [ ] Reviewer can confirm, edit, or reject
- [ ] Confirmed events published with verificationStatus = 'community_verified'
- [ ] Corrections feed back to improve extraction
- [ ] Trusted contributors can skip queue

---

### BNDY-010: Venue claim flow

**Status:** Ready
**Priority:** P1
**Phase:** 2

### Why

Community builders need to own their venues. Claimed venues have higher trust.

### Outcome

Venue owners can claim and verify their venue profiles.

### Acceptance Criteria

- [ ] User can request to claim a venue
- [ ] Verification options: email domain, document, manual review
- [ ] Claimed venue shows "Verified" badge
- [ ] Claimant can edit venue details
- [ ] Claimant can submit events directly (skip queue)

---

### BNDY-011: Artist claim flow

**Status:** Ready
**Priority:** P1
**Phase:** 2

### Why

Artists need to own their profiles. Claimed artists can confirm events.

### Outcome

Artists/bands can claim and verify their profiles.

### Acceptance Criteria

- [ ] User can request to claim an artist
- [ ] Verification options: social link, document, manual review
- [ ] Claimed artist shows "Verified" badge
- [ ] Claimant can edit artist details
- [ ] Claimant can confirm/deny events they're listed on

---

## Phase 3: Relationship Graph

Start capturing relationships between entities.

---

### BNDY-012: Add relationship records

**Status:** Needs design
**Priority:** P2
**Phase:** 3

### Why

The relationship graph is the real asset. This is what makes bndy smarter than Google.

### Outcome

System tracks relationships between entities.

### Acceptance Criteria

- [ ] Relationship entity created (see [[05-entities/relationship-model]])
- [ ] artist_played_venue relationships auto-created from events
- [ ] artist_played_event relationships auto-created
- [ ] Relationships have confidence and strength scores
- [ ] Relationships track firstSeen/lastSeen/occurrenceCount

---

### BNDY-013: Artist-to-venue history

**Status:** Needs design
**Priority:** P2
**Phase:** 3

### Why

Knowing which artists play which venues enables booking recommendations.

### Outcome

System shows venue booking history for artists.

### Acceptance Criteria

- [ ] Artist profile shows venues played
- [ ] Venue profile shows artists booked
- [ ] Frequency and recency visible
- [ ] "Artists like X who play here" query possible

---

### BNDY-014: Artist similarity

**Status:** Needs design
**Priority:** P2
**Phase:** 3

### Why

Similarity enables "if you like X, you'll like Y" recommendations.

### Outcome

System computes artist similarity from shared venues and genres.

### Acceptance Criteria

- [ ] artist_similar_to relationships computed
- [ ] Similarity based on: shared venues, genre overlap, audience overlap
- [ ] "Similar artists" section on artist pages
- [ ] API endpoint for similar artists

---

## Phase 4: Scene Intelligence

Build higher-value features on top of the graph.

---

### BNDY-015: Venue recommendations for bands

**Status:** Future
**Priority:** P3
**Phase:** 4

"Where could my band get booked?" - using relationship graph to suggest realistic venues.

---

### BNDY-016: Band recommendations for venues

**Status:** Future
**Priority:** P3
**Phase:** 4

"Which bands should we book?" - using relationship graph to suggest artists.

---

### BNDY-017: Local scene pages

**Status:** Future
**Priority:** P3
**Phase:** 4

Curated pages for specific scenes (e.g., "Bristol jazz", "Leeds punk").

---

### BNDY-018: Natural language search

**Status:** Future
**Priority:** P3
**Phase:** 4

"Find me a proper indie gig near Stockport this Saturday" - semantic search over the graph.

---

## Related

- [[now]] - Current sprint
- [[later]] - Future ideas
- [[01-strategy/ai-native-reframe|AI-Native Reframe]]
