# Clarification Model

## Purpose

A Clarification Request represents ambiguity that requires human resolution.

Ambiguity is not an error. It's a natural part of understanding the world from incomplete evidence. The Brain must surface ambiguity explicitly and resolve it through conversation.

## Why First-Class Ambiguity?

Without explicit ambiguity handling:
- Silent wrong matches (The Rigger → wrong venue)
- Weak entities created and never strengthened
- Multiple duplicates for same real-world entity
- No path to resolution

With ClarificationRequests:
- Ambiguity is visible
- Resolution flows through chat naturally
- Resolved ambiguities strengthen confidence
- Learning loop closes

## TypeScript Interface

```typescript
interface ClarificationRequest {
  clarificationId: string;       // clar_xxxxxxxx

  // What needs clarifying (at least one must be set)
  entityCandidateId?: string;    // Which candidate is ambiguous
  claimId?: string;              // Which claim is ambiguous
  evidencePackId?: string;       // Which pack has ambiguity

  // The question
  question: string;              // Human-readable question
  questionType: ClarificationQuestionType;

  // Options if applicable
  options?: ClarificationOption[];

  // Context for the user
  context?: ClarificationContext;

  // Resolution
  status: ClarificationStatus;
  resolvedBy?: string;           // User ID
  resolution?: string;           // What they chose or typed
  selectedOptionId?: string;     // If they picked an option
  resolvedAt?: string;

  // Effects of resolution
  resultingActions?: ClarificationAction[];

  createdAt: string;
  expiresAt?: string;            // Auto-dismiss if stale
}

type ClarificationQuestionType =
  | 'entity_match'        // "Is this The Rigger in Newcastle?"
  | 'date_confirm'        // "Is this May 15th 2026 or 2027?"
  | 'venue_location'      // "Where is this venue?"
  | 'artist_identity'     // "Is this the same Stingray?"
  | 'event_duplicate'     // "Is this the same event as...?"
  | 'claim_validity';     // "Does this claim look correct?"

type ClarificationStatus =
  | 'open'                // Awaiting resolution
  | 'resolved'            // User answered
  | 'dismissed'           // User skipped
  | 'expired'             // Timed out
  | 'superseded';         // Resolved by other means

interface ClarificationOption {
  optionId: string;              // opt_xxxxxxxx
  label: string;                 // "The Rigger, Newcastle-under-Lyme"
  description?: string;          // "Pub on High Street, capacity 150"
  entityId?: string;             // vnue_123 if selecting existing entity
  confidence?: number;           // 0.0-1.0, how likely this is correct
  isCreateNew?: boolean;         // True if this option creates new entity
}

interface ClarificationContext {
  signalId?: string;             // Source signal
  rawText?: string;              // Relevant text from signal
  relatedEntityIds?: string[];   // Entities involved
  previousClarifications?: string[]; // Earlier resolutions
}

interface ClarificationAction {
  actionType: 'link_entity' | 'create_entity' | 'update_claim' | 'strengthen_pack';
  entityId?: string;
  claimId?: string;
  packId?: string;
  details?: string;
}
```

## ID Format

```
clar_[8-char-nanoid]

Examples:
clar_a1b2c3d4
clar_x9y8z7w6
```

## Lifecycle

```
Evidence arrives with ambiguity
↓
ClarificationRequest created
Status: OPEN
↓
Chat surfaces question naturally
"Is this The Rigger in Newcastle-under-Lyme?"
↓
User responds
↓
Resolution recorded
Status: RESOLVED
↓
Effects applied:
- Entity linked or created
- Pack strength updated
- Claim confidence increased
```

## Generation Triggers

ClarificationRequests are generated when:

| Trigger | Example |
|---------|---------|
| Multiple entity matches | "The Rigger" matches 2 venues |
| Low confidence claim | Date inferred with year uncertain |
| Missing required data | Venue mentioned but not found |
| Potential duplicate | Similar event already exists |
| Conflicting evidence | Two signals disagree on time |

## Example Flow

### 1. Signal Arrives

```
"STINGRAY LIVE AT THE RIGGER THURSDAY 15TH MAY 8PM"
```

### 2. Claims Generated

```
venue_hosts: "The Rigger" → ???
```

### 3. Entity Search Finds Ambiguity

```
Matches:
- vnue_123: The Rigger, Newcastle-under-Lyme
- vnue_456: The Rigger, Sheffield
```

### 4. ClarificationRequest Created

```typescript
{
  clarificationId: "clar_abc12345",
  claimId: "clm_xyz789",
  question: "Which venue called 'The Rigger' is this?",
  questionType: "entity_match",
  options: [
    {
      optionId: "opt_001",
      label: "The Rigger, Newcastle-under-Lyme",
      description: "Pub on High Street",
      entityId: "vnue_123",
      confidence: 0.65
    },
    {
      optionId: "opt_002",
      label: "The Rigger, Sheffield",
      description: "Music venue on Ecclesall Road",
      entityId: "vnue_456",
      confidence: 0.35
    },
    {
      optionId: "opt_new",
      label: "This is a different venue",
      isCreateNew: true
    }
  ],
  status: "open",
  createdAt: "2026-05-04T10:00:00Z"
}
```

### 5. Chat Surfaces Naturally

```
bndy:
I found an event for Stingray at "The Rigger" on May 15th.

I know two venues called The Rigger - is this the one
in Newcastle-under-Lyme or Sheffield?

[Newcastle-under-Lyme] [Sheffield] [Neither - it's a new venue]
```

### 6. User Resolves

User clicks "Newcastle-under-Lyme"

### 7. Resolution Applied

```typescript
{
  status: "resolved",
  resolvedBy: "user_123",
  selectedOptionId: "opt_001",
  resolution: "The Rigger, Newcastle-under-Lyme",
  resolvedAt: "2026-05-04T10:01:00Z",
  resultingActions: [
    {
      actionType: "link_entity",
      claimId: "clm_xyz789",
      entityId: "vnue_123"
    },
    {
      actionType: "strengthen_pack",
      packId: "pack_event123"
    }
  ]
}
```

## DynamoDB Access Patterns

| Pattern | PK | SK |
|---------|----|----|
| Get clarification | `CLAR#clar_xxx` | `#METADATA` |
| List open for user | GSI: `STATUS#open` | `CLAR#clar_xxx` |
| List by entity | GSI: `ENTITY#vnue_xxx` | `CLAR#clar_xxx` |
| List by pack | GSI: `PACK#pack_xxx` | `CLAR#clar_xxx` |

## Chat Integration

The chat interface should:

1. **Surface clarifications naturally** - Not as a separate queue, but in conversation
2. **Provide context** - Show the signal text, related entities
3. **Make resolution easy** - Buttons for options, text for custom answers
4. **Confirm the effect** - "Great, I linked this event to The Rigger in Newcastle"

## Resolution Effects

When a clarification is resolved:

1. **Entity linking** - Claim gets linked to chosen entity
2. **Pack strengthening** - Evidence pack confidence increases
3. **Verification promotion** - Entity verification may upgrade
4. **Future matching** - System learns this association

## Expiration

Clarifications can expire if:
- Signal becomes irrelevant (event date passed)
- Pack gets resolved by other evidence
- Entity gets merged

Expired clarifications are marked `superseded` or `expired`.

## Related

- [[evidence-pack-model]] - Packs can have ambiguities
- [[claim-model]] - Claims can trigger clarifications
- [[../10-brain/signal-to-claim-model]] - How claims surface uncertainty
- [[../03-backlog/next-5-phases]] - Phase C: Ambiguity Resolution

