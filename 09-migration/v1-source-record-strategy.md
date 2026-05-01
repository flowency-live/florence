# v1 Source Record Strategy

## Purpose

The existing bndy platform already contains valuable grassroots music data.

This data should enter the future platform through the same provenance-aware pipeline as future scraped, submitted and AI-extracted records.

## Core Principle

v1 data becomes a source.

Not special-case truth.

## Migration Source Type

Every migrated v1 record should generate a SourceRecord.

```typescript
{
  sourceType: 'migration',
  sourceName: 'bndy_v1',
  extractionMethod: 'manual'
}
```

## Example Flow

```text
Existing bf_events record
↓
Create SourceRecord
↓
Generate canonical event candidate
↓
Resolve canonical venue
↓
Resolve canonical artists
↓
Assign confidence
↓
Publish canonical event
```

## Migration Metadata

```typescript
migrationMetadata: {
  migratedFrom: 'bndy_v1',
  originalEntityType: 'event',
  originalId: 'abc123',
  migrationBatch: '2026-05-01'
}
```

## Confidence Defaults

| Entity | Suggested Base Confidence |
|---|---|
| Venue with coordinates + events | 0.85 |
| Venue missing address | 0.60 |
| Artist with multiple linked events | 0.80 |
| Sparse artist record | 0.55 |
| Event with linked venue + artist | 0.85 |
| Event missing venue linkage | 0.50 |

## Important Rule

Migration is only one contributing source.

Future sources may:
- corroborate
- correct
- enrich
- invalidate
- supersede

existing migrated data.

## Related

- [[v1-to-canonical-plan]]
- [[canonical-id-strategy]]
- [[../05-entities/source-record-model|Source Record Model]]
