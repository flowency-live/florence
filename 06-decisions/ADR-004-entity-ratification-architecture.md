# ADR-004: Entity Ratification Architecture

---

## Status

**Status:** Accepted
**Date:** 03/05/2026
**Deciders:** Jason, Claude
**Supersedes:** None

## Context

BUILD-004 implemented entity resolution that auto-links claims to entities based on fuzzy name matching. This creates several problems:

- **Blind matching**: "The Rigger" auto-matches without confirming which Rigger (there are 3+ in the UK)
- **Claims as UI**: Current flow requires users to see and review individual claims, which is not conversational
- **External refs as truth**: If we add Facebook/Google/Gigantic links, they could override BNDY's canonical identity
- **No ambiguity handling**: System either matches or creates, never asks "which one?"
- **Schema violations**: Event drafts are created with empty required fields, cast through `unknown`

The fundamental architectural question: **Who decides what's true?**

Options:
1. **Auto-match with high confidence threshold** - Machine decides, human reviews exceptions
2. **Claims admin UI** - Human reviews every claim explicitly
3. **Conversational ratification** - Machine proposes candidates, chat asks human to confirm/correct

Forces at play:
- **User experience**: People want to chat, not admin claims
- **Accuracy**: Local venues have name collisions (The Crown, The Rigger, The George)
- **Provenance**: Need to know why we believe something
- **Scalability**: Can't require human review of every field
- **Trust**: Published entities should be defensible

## Decision

We decided to use **Conversational Entity Ratification** because:

> In the context of building a knowledge graph for grassroots live music,
> facing name collisions, external ref proliferation, and user experience expectations,
> we decided to make claims internal evidence and use chat for human ratification,
> to achieve a conversational UX with defensible knowledge provenance,
> accepting that entity resolution is slower and requires conversational context.

### Core Principles

| Principle | Implication |
|-----------|-------------|
| **Claims are internal** | Users never see "claims". They see "Here's what I found" |
| **BNDY IDs are canonical** | `vnue_abc12345` is THE Rigger. External refs are evidence. |
| **External refs are evidence** | Google Place ID, Facebook Page ID, Gigantic slug → linked as evidence, not identity |
| **Candidates, not auto-match** | "I found 3 venues called The Rigger. Which one?" |
| **Chat is ratification** | Human confirms/corrects via conversation, not claims UI |
| **Graph is durable** | Once ratified, entity relationships are knowledge, not pending claims |

### Entity Resolution Flow

```
Signal (poster/text)
    ↓
Claims generated (INTERNAL - never shown)
    ↓
Brain creates EntityCandidates
    ├── Venue: The Rigger [3 matches found]
    ├── Artist: Stingray [1 match, high confidence]
    └── Event: new draft
    ↓
Chat asks follow-up questions
    "Which Rigger? Newcastle-under-Lyme, Bristol, or London?"
    ↓
Human confirms/corrects
    ↓
Graph updated with ratified entities
    ↓
Claims remain internal evidence
```

### Key Schema Changes Required

1. **EntityCandidate** - Proposed entity match with confidence
2. **RatificationSession** - Tracks conversation state for entity confirmation
3. **ExternalRef** - Links to Google/Facebook/ticketing with provenance
4. **Event verificationStatus** - Track trust level separate from existence

### Event Creation Clarification

Events CAN be created from a single signal. What changes is the verification level, not whether the event can exist.

```text
Evidence creates possibility.
Trust determines confidence.
Corroboration strengthens certainty over time.
```

**Trusted submitter** (verified venue, claimed artist, known promoter):
```
submit signal → interpretation → event created immediately
verificationStatus: 'submitter_verified' | 'venue_confirmed' | 'artist_confirmed'
```

**Unknown/low-trust submitter** (anonymous, scraped content):
```
submit signal → interpretation → proposed/draft event
verificationStatus: 'unverified' | 'community_verified' | 'source_correlated'
```

This is critical for grassroots live music. Many legitimate gigs:
- are not on Google
- are not on ticketing sites
- may only exist as a poster, Instagram story, or chat message from the organiser

BNDY should not require external corroboration before allowing those events to exist in the graph. That is part of the moat.

## Consequences

### Positive

- **Conversational UX**: Users chat, don't admin claims
- **Defensible knowledge**: Every entity has provenance trail
- **Handles ambiguity**: "Which Rigger?" is a natural question
- **External refs as evidence**: Facebook page change doesn't break identity
- **Clean separation**: Claims are internal, entities are external
- **Schema safety**: No more `unknown` casts

### Negative

- **Slower resolution**: Requires conversation roundtrips
- **Context dependency**: Need session state for multi-turn ratification
- **More complex**: EntityCandidate + RatificationSession + ExternalRef models
- **Migration**: BUILD-004 code needs significant rework

### Neutral

- Claims still exist and are stored - they're just not shown
- Entity matching still uses fuzzy name + location - just returns candidates
- Can still auto-confirm high-confidence single matches

## Alternatives Considered

### Option 1: Claims Admin UI

Keep claims visible, build better review interface.

**Pros:**
- Simple model
- Full transparency
- Already partially built

**Cons:**
- Not conversational
- Users don't want to "review claims"
- Scales poorly

**Why rejected:** User experience. Nobody wants to admin claims.

### Option 2: Auto-match Everything

Trust the algorithm, let users correct mistakes later.

**Pros:**
- Fast
- No friction
- Simple

**Cons:**
- Creates bad data
- Name collisions cause silent errors
- No provenance for decisions

**Why rejected:** Data quality. Silent failures are worse than asking.

### Option 3: Confidence Threshold with Manual Fallback

Auto-match above 0.95 confidence, manual review below.

**Pros:**
- Best of both worlds
- Handles easy cases automatically
- Only asks when uncertain

**Cons:**
- Confidence scores are unreliable for names
- "The Rigger" might score 0.99 against wrong Rigger
- Location context is critical, not just name

**Why rejected:** Confidence without context is dangerous. Need location-aware matching.

## Implementation Phases

### Phase 1: Fix BUILD-004 Safety Issues

- Remove event draft `unknown` cast
- Use edited claim values in resolution
- Return candidates instead of auto-linking when ambiguous
- Add location-aware venue matching

### Phase 2: EntityCandidate Model

- Schema for candidate entities with match scores
- Session tracking for multi-turn ratification
- API for chat-driven confirmation

### Phase 3: Conversational Ratification

- Chat layer asks follow-up questions
- ExternalRef model for linked evidence
- Graph update on confirmation

## Related

- [[../03-backlog/now|Now]] - BUILD status
- [[../05-entities/claimed-entity-model|Claimed Entity Model]] - How entities work
- [[../10-brain/signal-to-claim-model|Signal to Claim Model]] - Claims as evidence
- [[../10-brain/bndy-brain-concept|Brain Concept]] - The reasoning layer
- [[ADR-003-signal-processing-stack|ADR-003]] - Signal processing decisions
