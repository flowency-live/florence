# Derived Fields and Field Corruption

## Purpose

This note captures a critical design finding from the v1/v2 transition.

In the old model, AI agents and APIs were asked to update structured fields directly, such as:

```text
artist.location = "North West UK"
artist.genre = "rock"
```

That is brittle.

It caused a real failure mode: regional artist lists were written into a specific location field. The application then tried to geocode regional values such as "North West UK" or "North East" as if they were towns/cities, producing incorrect location suggestions and polluted artist records.

## Design Finding

Agents should not write ambiguous real-world knowledge directly into brittle product fields.

They should generate claims, supported by evidence.

Product fields should be derived views, not the primary truth.

## Old Model

```text
Agent sees "North West bands"
↓
Agent writes location = "North West UK"
↓
Application treats it as a specific place
↓
Geocoder resolves incorrectly
↓
Artist record is corrupted
```

## New Model

```text
Agent sees "North West bands"
↓
Agent creates a claim:
  Artist is associated with the North West England music scene
↓
Evidence is retained:
  Imported from regional source list
↓
Knowledge graph stores regional association
↓
UI derives the appropriate display/search field
```

## Location Example

Do not treat all location-like strings as specific places.

### Bad

```typescript
artist.location = "North West UK";
```

### Better

```typescript
Claim {
  claimType: 'artist_region_association',
  subject: 'artist_x',
  predicate: 'associated_with_region',
  object: 'North West England',
  reasoning: 'Artist appeared in a North West regional source list',
  evidence: ['signal_x']
}
```

Then the product can derive:

```typescript
artistView = {
  homeTown: null,
  primaryRegion: 'North West England',
  locationDisplay: 'North West England',
  locationType: 'regional'
}
```

## Genre Example

Do not force all genre/style information into a single genre field.

### Bad

```typescript
artist.genre = 'rock';
```

### Better

```typescript
Claim {
  claimType: 'artist_style_signal',
  subject: 'artist_x',
  predicate: 'described_as',
  object: 'classic rock covers',
  reasoning: 'Facebook bio describes the band as classic rock covers',
  evidence: ['signal_y']
}
```

The system can later derive:

```typescript
artistView = {
  primaryGenre: 'rock',
  subGenres: ['classic rock', 'covers'],
  sceneTags: ['pub rock', 'covers circuit']
}
```

## Claims Before Fields

Agents should create or update claims for ambiguous knowledge categories:

- region
- hometown
- service area
- scene
- genre
- subgenre
- style
- audience fit
- venue fit
- promoter association
- artist similarity
- recurring night association

Only stable, strongly evidenced facts should become direct fields.

## Direct Fields Still Exist

The new model does not remove fields.

It changes their role.

Fields are needed for:

- UI display
- search indexes
- map filtering
- API responses
- performance
- compatibility with v1/v2 clients

But these fields should usually be derived from accepted claims and relationships.

## Derived View Pattern

```text
Evidence
↓
Claims
↓
Accepted knowledge graph
↓
Derived read model
↓
UI/API fields
```

Example read model:

```typescript
ArtistReadModel {
  artistId: string;
  displayName: string;
  locationDisplay?: string;
  locationType?: 'specific' | 'regional' | 'national' | 'unknown';
  homeTown?: string;
  primaryRegion?: string;
  serviceAreas?: string[];
  primaryGenre?: string;
  subGenres?: string[];
  sceneTags?: string[];
}
```

## Guardrail for Agents and MCP Tools

No agent or MCP tool should expose only a generic ambiguous field like:

```typescript
location: string
```

For location, the tool must distinguish at least:

```typescript
locationType: 'specific' | 'regional' | 'national' | 'unknown'
locationValue: string
```

Or better, write a claim rather than a field mutation.

## Rule

If the source tells us something contextual, cultural, regional or interpretive, store it as knowledge first.

Do not prematurely collapse it into a UI field.

## Related

- [[signal-to-claim-model]]
- [[claim-evidence-graph]]
- [[memory-and-reasoning-model]]
- [[../05-entities/artist-model|Artist Model]]
- [[../09-migration/v1-to-canonical-plan|v1 to Canonical Plan]]
