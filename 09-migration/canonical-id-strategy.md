# Canonical ID Strategy

## Purpose

Canonical IDs allow bndy to:

- merge duplicate records
- track provenance
- maintain stable references
- attach relationships over time
- survive source changes and renames

The canonical ID becomes the long-lived identity for an entity.

## Core Principle

Source IDs are temporary.

Canonical IDs are permanent.

## ID Formats

### Venue

```text
vnue_<generated>
```

Example:

```text
vnue_8f4ab2
```

### Artist

```text
arts_<generated>
```

Example:

```text
arts_72dca1
```

### Event

```text
evnt_<generated>
```

Example:

```text
evnt_1bc883
```

### Relationship

```text
rel_<generated>
```

### Source Record

```text
src_<generated>
```

## Never Use External IDs As Canonical IDs

Do not use:
- Facebook IDs
- Dice IDs
- Eventbrite IDs
- Skiddle IDs
- v1 Dynamo IDs

These should be stored as source references or legacy references.

## Legacy ID Preservation

Migrated records should preserve their original identifiers.

```typescript
legacyIds: {
  bndyV1?: string;
  facebook?: string;
  skiddle?: string;
}
```

## Alias Strategy

Canonical entities can have many aliases.

Example:

```typescript
aliases: [
  'The Kings Arms',
  'Kings Arms',
  'King\'s Arms Stoke'
]
```

Aliases are critical for:
- duplicate detection
- fuzzy matching
- source reconciliation
- search

## Merge Rules

When duplicate entities are identified:

1. Keep the oldest or highest-confidence canonical ID
2. Merge aliases
3. Merge source references
4. Preserve all legacy IDs
5. Redirect relationships to surviving canonical entity

Never destroy provenance.

## Event Identity

Events are harder because they are temporal.

Canonical event identity should primarily derive from:

```text
venue + date/time + artists + title similarity
```

The same event from multiple sources should collapse into one canonical event.

## Confidence and Identity

Identity resolution should always produce:

```typescript
{
  matchedCanonicalId?: string;
  confidence: number;
  suggestedAction: 'link' | 'create' | 'review';
}
```

Do not auto-merge low-confidence matches.

## Related

- [[v1-to-canonical-plan]]
- [[../05-entities/venue-model|Venue Model]]
- [[../05-entities/artist-model|Artist Model]]
- [[../05-entities/event-model|Event Model]]
