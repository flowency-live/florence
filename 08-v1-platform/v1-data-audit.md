# v1 Data Audit

## Purpose

Before redesigning ingestion or relationships, understand the quality and shape of the existing bndy data.

The existing platform already contains:
- artists
- venues
- events
- relationships hidden implicitly in event participation
- geographic clustering
- behavioural patterns

This is seed data for the canonical intelligence layer.

## Audit Goals

1. Understand current data quality
2. Measure duplicate rates
3. Identify high-confidence seed records
4. Identify migration risks
5. Determine canonicalisation effort

## Entity Audit Areas

### Venues

Assess:
- duplicate venues
- missing coordinates
- postcode quality
- inconsistent naming
- inactive venues
- venue density by city

### Artists

Assess:
- duplicate artists/bands
- missing links/socials
- sparse records
- alias problems
- disconnected artists with no events

### Events

Assess:
- orphaned events
- duplicate events
- invalid dates
- stale events
- missing venue links
- missing artist links

## Recommended Outputs

### Summary Metrics

```text
Total venues
Total artists
Total events
Duplicate candidates
Records missing coordinates
Records missing venue linkage
Records missing artist linkage
```

### Quality Bands

Classify entities:

| Band | Meaning |
|---|---|
| High confidence | Good candidate for canonical seed |
| Medium confidence | Needs enrichment |
| Low confidence | Needs review or archive |

## Migration Opportunity

The existing data should not be viewed as technical debt alone.

It is:
- training data
- matching data
- geographic grounding
- relationship history
- source reliability evidence

## Recommended Next Technical Step

Build a migration pipeline that:

```text
Reads v1 records
↓
Creates migration source records
↓
Generates canonical candidates
↓
Runs duplicate detection
↓
Publishes canonical entities
```

## Related

- [[../09-migration/v1-to-canonical-plan|v1 to Canonical Plan]]
- [[architecture]]
- [[data-models]]
