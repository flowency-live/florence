# Claimed Entity Model

## Purpose

Artists and venues still need to own and manage their presence in bndy.

The AI-native model does not remove conceptual entities. It changes what they are.

Entities stop being simple CRUD records and become canonical knowledge objects with ownership, evidence, relationships and derived views.

## Core Principle

```text
Entities remain.
Dumb fields do not remain as the source of truth.
```

bndy still has:

- Artist
- Venue
- Event
- Promoter
- Agent
- Community Builder
- Organisation

But each entity is now layered.

## Layered Entity Pattern

```text
Canonical Entity
├── Claimed profile
├── Owner-managed facts
├── Submitted events
├── Claims and evidence
├── Relationships
├── Derived read model
└── Moderation / challenge history
```

## Claimed Profile Data

Entity owners should be able to directly manage stable profile information.

For artists:

- display name
- bio
- images
- website
- social links
- music links
- booking contact
- availability notes
- press assets

For venues:

- display name
- description
- photos
- website
- address/contact details
- opening information
- booking contact
- capacity information
- accessibility notes

This information is owner-contributed and should be treated as high-quality, but it is still provenance-aware.

## Knowledge-Backed Data

Some information should not be overwritten directly by an owner or agent.

Examples:

- event history
- venues played
- artists booked
- associated regions/scenes
- inferred genre/style
- promoter relationships
- artist similarity
- venue-fit signals
- audience-fit signals
- source corroboration

This belongs in claims, evidence and relationships.

Owners can challenge or correct it, but they should not silently erase it.

## Entity Ownership vs Entity Identity

Ownership is separate from identity.

A canonical artist or venue can exist before it is claimed.

A claim gives a person or organisation permission to manage a profile layer, not ownership of the underlying truth.

```typescript
interface EntityClaim {
  claimId: string;
  entityType: 'artist' | 'venue' | 'event' | 'promoter' | 'series';
  entityId: string;

  claimantType: 'user' | 'organisation';
  claimantId: string;

  role: EntityRole;
  status: 'requested' | 'approved' | 'rejected' | 'revoked';

  verificationMethod?: 'email_domain' | 'social_proof' | 'document' | 'manual_review' | 'existing_relationship';
  verifiedAt?: string;
  verifiedBy?: string;

  createdAt: string;
  updatedAt: string;
}

type EntityRole =
  | 'owner'
  | 'manager'
  | 'booking_agent'
  | 'promoter'
  | 'editor'
  | 'viewer';
```

## Organisations and Agents

bndy needs organisations because one user may operate across many entities.

Examples:

- an agent managing multiple acts
- a promoter managing several event series
- a venue group managing multiple venues
- a community builder operating across a local scene
- a booking agency representing artists and venues

```typescript
interface Organisation {
  organisationId: string;
  name: string;
  organisationType:
    | 'promoter'
    | 'booking_agent'
    | 'venue_group'
    | 'artist_management'
    | 'community_group'
    | 'media'
    | 'other';

  members: OrganisationMember[];
  managedEntities: ManagedEntity[];

  createdAt: string;
  updatedAt: string;
}

interface OrganisationMember {
  userId: string;
  role: 'owner' | 'admin' | 'editor' | 'viewer';
}

interface ManagedEntity {
  entityType: 'artist' | 'venue' | 'event' | 'promoter' | 'series';
  entityId: string;
  role: EntityRole;
}
```

## Event Submission by Owners and Agents

Claimed entities can submit events directly.

However, submitted events still become signals and claims.

```text
Owner submits event
↓
Signal created
↓
Claims generated
↓
Event draft proposed
↓
Higher trust because source is owner/agent
↓
Canonical event published or queued for review
```

Owner submission is not outside the intelligence model. It is a high-trust signal inside it.

## Product Rule

Owners can manage profiles.

The brain manages knowledge.

```text
Profile data = editable by authorised owners/agents
Knowledge data = claim/evidence backed, challengeable, explainable
Read model = derived from both
```

## Example: Artist

```text
Canonical Artist: arts_stingray
├── Claimed profile
│   ├── bio
│   ├── images
│   ├── Facebook URL
│   └── booking contact
├── Knowledge
│   ├── associated_with_region: North West England
│   ├── has_style_signal: classic rock covers
│   ├── played_at: The Rigger
│   └── similar_to: other local rock/covers acts
└── Derived view
    ├── locationDisplay: North West England
    ├── primaryGenre: rock
    └── sceneTags: pub rock, covers circuit
```

## Example: Venue

```text
Canonical Venue: vnue_the_rigger
├── Claimed profile
│   ├── venue description
│   ├── photos
│   ├── booking email
│   └── opening/contact information
├── Knowledge
│   ├── hosts_genre: rock
│   ├── books_artist: Stingray
│   ├── associated_scene: Newcastle-under-Lyme rock circuit
│   └── source_corrobation from venue site, ticketing links and posters
└── Derived view
    ├── venueType: pub / live music venue
    ├── primaryGenres: rock, covers
    └── upcomingEvents
```

## Guardrail

A profile edit should not directly mutate the knowledge graph without creating evidence.

If an owner changes contextual information such as genre, region or scene, bndy should create a claim:

```text
Claim: Artist self-describes as blues rock
Evidence: owner profile edit
Source strength: owner asserted
Status: accepted unless challenged
```

## Related

- [[artist-model]]
- [[venue-model]]
- [[event-model]]
- [[community-builder-model]]
- [[../10-brain/derived-fields-and-field-corruption|Derived Fields and Field Corruption]]
- [[../10-brain/claim-evidence-graph|Claim-Evidence Graph]]
