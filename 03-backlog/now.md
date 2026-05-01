# Now

**Phase 1: Build the Intelligence Underneath**

Keep the map, but build the canonical identity and provenance layer that makes bndy defensible.

---

## BNDY-001: Add provenance to every event

**Status:** Ready
**Priority:** P0
**Phase:** 1

### Why

Without provenance, we're just another listing site. With provenance, we become a trust-aware intelligence system.

### Outcome

Every event knows where it came from and how reliable it is.

### Acceptance Criteria

- [ ] Event has sourceIds[] array
- [ ] Event has extractionConfidence score
- [ ] Event has verificationStatus
- [ ] Event has provenanceSummary
- [ ] Event has uncertaintyFlags[]
- [ ] SourceRecord entity created (see [[05-entities/source-record-model]])

---

## BNDY-002: Create canonical venue IDs

**Status:** Ready
**Priority:** P0
**Phase:** 1

### Why

Venues are the anchor entity. Canonical identity enables deduplication, trust, and relationship tracking.

### Outcome

Every venue has a canonical ID with aliases, sources, and confidence scoring.

### Acceptance Criteria

- [ ] Venue has canonical ID (vnue_xxx format)
- [ ] Venue can have aliases (alternative names)
- [ ] Venue can have source references
- [ ] Venue has confidenceScore (0-1)
- [ ] Venue has verificationStatus
- [ ] Duplicate detection matches existing venues (4-level from v1)

### Notes

See [[05-entities/venue-model]] and [[08-v1-platform/data-models]] for v1 compatibility.

---

## BNDY-003: Create canonical artist IDs

**Status:** Ready
**Priority:** P0
**Phase:** 1

### Why

Artists are the second key entity. Canonical identity enables relationship tracking and similarity.

### Outcome

Every artist has a canonical ID with aliases, sources, and confidence scoring.

### Acceptance Criteria

- [ ] Artist has canonical ID (arts_xxx format)
- [ ] Artist can have aliases (stage names, band variants)
- [ ] Artist can have source references
- [ ] Artist has confidenceScore (0-1)
- [ ] Artist has verificationStatus
- [ ] Duplicate detection matches existing artists

---

## BNDY-004: Add duplicate detection workflow

**Status:** Ready
**Priority:** P0
**Phase:** 1

### Why

Without deduplication, data quality degrades. Users see the same event multiple times.

### Outcome

System detects duplicates and queues them for merge or human review.

### Acceptance Criteria

- [ ] Venue duplicate detection (coordinates + name similarity)
- [ ] Artist duplicate detection (name + genre overlap)
- [ ] Event duplicate detection (venue + date + name)
- [ ] Admin queue for uncertain duplicates
- [ ] Merge workflow preserves best data from both records
- [ ] Source references merged on duplicate resolution

---

## BNDY-005: Add trust/confidence scores

**Status:** Ready
**Priority:** P0
**Phase:** 1

### Why

Not all data is equal. Confidence scores let AI act on probability and escalate uncertainty.

### Outcome

Every entity has a confidence score that reflects data quality.

### Acceptance Criteria

- [ ] Venue confidenceScore computed from sources + verification
- [ ] Artist confidenceScore computed from sources + verification
- [ ] Event confidenceScore computed from extraction + entity resolution
- [ ] Low-confidence entities flagged for human review
- [ ] UI shows verification badges

---

## BNDY-006: Add source reliability reporting

**Status:** Ready
**Priority:** P1
**Phase:** 1

### Why

Track which sources are reliable over time. Prioritise good sources, deprioritise bad ones.

### Outcome

Dashboard showing source quality metrics.

### Acceptance Criteria

- [ ] Track successRate per source
- [ ] Track correctionRate per source
- [ ] Track duplicateRate per source
- [ ] Admin can see source reliability dashboard
- [ ] Low-reliability sources flagged

---

## Related

- [[next]] - Phase 2-4 work
- [[later]] - Future ideas
- [[risks]] - Known risks
- [[01-strategy/bndy-strategy|Strategy]] - Why we're doing this
