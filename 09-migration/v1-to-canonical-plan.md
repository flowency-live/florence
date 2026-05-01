# v1 to Canonical Migration Plan

## Purpose

bndy already has useful data in DynamoDB from the existing platform: artists, venues and events created through the traditional product model.

This data should not be discarded.

The migration goal is to turn existing v1 records into the seed dataset for the new canonical intelligence layer.

## Strategic Shift

We are moving from:

```text
User-created artist / venue / event records
```

To:

```text
Canonical entities with provenance, confidence, aliases, source history and relationships
```

The database may still be DynamoDB. The important change is not the storage engine. The important change is the meaning and shape of the records.

## Migration Principle

Treat v1 data as a source, not as final truth.

Every existing v1 record should be imported through the same provenance-aware model as future scraped, submitted or AI-extracted data.

```text
Existing DynamoDB record
↓
SourceRecord: bndy_v1 migration
↓
Canonical candidate
↓
Identity resolution / duplicate detection
↓
Canonical entity
↓
Relationship-ready record
```

## Source Classification

Existing v1 records should create source records with:

```typescript
sourceType: 'migration'
sourceName: 'bndy_v1'
extractionMethod: 'manual'
processingStatus: 'published'
```

Where useful, retain the original raw record payload for audit/debugging.

## Entity Treatment

### Venues

Existing venues become canonical venue candidates.

Preserve:
- original v1 ID
- venue name
- address/postcode
- coordinates
- source of creation if known
- linked events
- any existing metadata

Add:
- canonical ID
- aliases
- confidence score
- verification status
- source IDs
- legacy ID reference

### Artists

Existing artists/bands become canonical artist candidates.

Preserve:
- original v1 ID
- artist/band name
- social/music links
- linked events
- genre/style metadata if present

Add:
- canonical ID
- aliases
- confidence score
- verification status
- source IDs
- legacy ID reference

### Events

Existing events become canonical event candidates.

Preserve:
- original v1 ID
- venue reference
- artist references
- title/name
- date/time
- ticket/info links
- event status

Add:
- canonical venue ID
- canonical artist IDs
- source IDs
- event provenance
- confidence score
- uncertainty flags

## Legacy ID Strategy

Every migrated canonical entity should retain a pointer back to its v1 origin.

```typescript
legacyIds: {
  bndyV1?: string;
}
```

Do not reuse v1 IDs as canonical IDs unless they already meet the new ID convention.

## Do Not Big-Bang Migrate

Avoid a high-risk one-off migration.

Recommended approach:

1. Read existing v1 data
2. Create migration SourceRecords
3. Generate canonical candidates
4. Run duplicate detection
5. Create canonical records in parallel
6. Compare v1 and canonical outputs
7. Switch read paths gradually
8. Keep rollback path until confidence is high

## Migration Workstreams

### Workstream 1: Audit

Understand what exists now.

Outputs:
- record counts by entity type
- duplicate rates
- missing field analysis
- high-quality seed records
- problematic records

### Workstream 2: Canonical Mapping

Map v1 fields to future canonical fields.

Outputs:
- venue mapping
- artist mapping
- event mapping
- unmapped field list
- migration assumptions

### Workstream 3: Candidate Generation

Create canonical candidates from v1 records.

Outputs:
- candidate venues
- candidate artists
- candidate events
- migration source records
- confidence scores

### Workstream 4: Deduplication

Identify duplicates before treating records as canonical.

Outputs:
- duplicate venue candidates
- duplicate artist candidates
- duplicate event candidates
- merge recommendations

### Workstream 5: Parallel Run

Run v1 and canonical records side by side.

Outputs:
- comparison report
- missing record report
- confidence thresholds
- cutover decision

## Immediate Next Step

Create a v1 data audit from the current DynamoDB tables/collections before making structural changes.

## Related

- [[../08-v1-platform/v1-data-audit|v1 Data Audit]]
- [[canonical-id-strategy]]
- [[v1-source-record-strategy]]
- [[../05-entities/source-record-model|Source Record Model]]
- [[../04-architecture/data-model|Data Model]]
