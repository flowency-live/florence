# Event Verification Model

---

## Core Principle

```text
A single signal can create an event.

What changes is the verification/confidence level,
not whether the event can exist.
```

---

## Separate Concerns

The system separates:

| Question | Concern |
|----------|---------|
| Can this event exist? | **Existence** |
| How strongly do we trust this event? | **Verification** |

These are different concerns handled by different fields.

---

## Event Lifecycle Status

```typescript
eventStatus:
  | 'proposed'    // Initial submission, minimal validation
  | 'draft'       // Being refined, may have gaps
  | 'confirmed'   // Verified to exist
  | 'cancelled'   // Was confirmed, now cancelled
  | 'historical'  // Past event, archived
```

---

## Verification Status

```typescript
verificationStatus:
  | 'unverified'          // Unknown submitter, no corroboration
  | 'submitter_verified'  // Submitter has verified account
  | 'community_verified'  // Multiple users confirm
  | 'source_correlated'   // Found on external source
  | 'venue_confirmed'     // Venue owner/rep confirmed
  | 'artist_confirmed'    // Artist/band confirmed
```

---

## Creation Flows

### Trusted Submitter

Examples:
- Verified venue owner
- Claimed artist profile
- Known promoter/community builder

```text
submit signal
→ interpretation generates claims
→ event created immediately
→ verificationStatus reflects trusted source
```

### Unknown / Low-Trust Submitter

Examples:
- Anonymous user
- Random screenshot upload
- Scraped content

```text
submit signal
→ interpretation generates claims
→ proposed/draft event created
→ verificationStatus: 'unverified'
→ waits for corroboration or ratification
```

---

## Why This Matters

Grassroots live music is fragmented.

Many legitimate gigs:
- are not on Google
- are not on ticketing sites
- are not on venue websites
- may only exist as:
  - a poster
  - an Instagram story
  - a Facebook post
  - a chat message from the organiser

BNDY should be capable of becoming the first system to know about these events.

Requiring external corroboration before allowing events to exist would eliminate BNDY's potential moat.

---

## Entity Resolution Guidance

### Artists

```text
No exact match → create draft artist (acceptable)
```

### Venues

```text
No exact match → prefer candidate/proposed venue unless location confidence is strong
Multiple exact matches → return candidates for ratification
```

### Events

```text
May be created from a single signal.

Verification level depends on:
- source trust
- corroboration
- human confirmation
- ownership/claimed profiles
```

---

## Relationship to Brain Architecture

This does NOT invalidate the Brain model.

The Brain still:
- stores evidence
- versions interpretations
- tracks reasoning
- allows challenges/corrections
- accumulates corroboration over time

The difference is:

```text
Event existence can begin from one trusted signal.
Confidence evolves afterward.
```

---

## Related

- [[../06-decisions/ADR-004-entity-ratification-architecture|ADR-004]] - Ratification architecture
- [[claimed-entity-model|Claimed Entity Model]] - How entities work
- [[canonical-entity|Canonical Entity]] - Schema definitions
