# Agentic Intake Loop

## Not a Pipeline

A pipeline processes data in sequence:

```
Input → Step 1 → Step 2 → Step 3 → Output
```

The agentic intake loop is a reasoning cycle:

```
Evidence → Interpret → Compare → Propose → Challenge → Update
     ↑                                              │
     └──────────────────────────────────────────────┘
```

It can loop back. It can pause. It can ask for help.

## The Agent's Questions

When evidence arrives, the agent does not run a parser.

It asks questions:

### 1. What does this appear to tell us?

> "This Facebook post appears to announce a live music event. I see an artist name, venue name, and date."

### 2. What claims can I generate?

> "I can claim: Stingray performs at The Rigger on 15 May 2026. I can also claim: The Rigger is a venue. I can also claim: Stingray is an artist."

### 3. Do I already know these things?

> "Looking at memory: The Rigger exists (high confidence, 47 prior events). Stingray exists (medium confidence, 3 prior events)."

### 4. Does this contradict anything?

> "No contradictions found. This appears to be a new event, not an update to existing."

### 5. Does this strengthen existing knowledge?

> "This adds another event to The Rigger's history. This adds another event to Stingray's history."

### 6. What should change in the world model?

> "Propose: Create new event. Link to existing venue The Rigger. Link to existing artist Stingray. Attach this signal as evidence."

### 7. How confident am I?

> "Venue match: very confident (exact name, location matches). Artist match: confident (name matches, genre fits). Date: confident (Thursday 15 May is valid for 2026)."

### 8. Should I ask a human?

> "No blockers. All claims are reasonably confident. Propose auto-acceptance."

## Agent States

```
┌─────────────┐
│  RECEIVED   │ Signal just arrived
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ INTERPRETING│ Agent reading the evidence
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  COMPARING  │ Checking against memory
└──────┬──────┘
       │
       ├─────────────────────────┐
       │                         │
       ▼                         ▼
┌─────────────┐          ┌─────────────┐
│  PROPOSING  │          │ CONFLICTED  │ Contradiction found
└──────┬──────┘          └──────┬──────┘
       │                         │
       ├─────────────────────────┘
       │
       ▼
┌─────────────┐
│   PENDING   │ Awaiting acceptance
└──────┬──────┘
       │
       ├─────────────────┬────────────────┐
       │                 │                │
       ▼                 ▼                ▼
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│  ACCEPTED   │  │ CHALLENGED  │  │  REJECTED   │
└─────────────┘  └──────┬──────┘  └─────────────┘
                        │
                        ▼
                 ┌─────────────┐
                 │  RESOLVED   │ Human decided
                 └─────────────┘
```

## The Interpretation Phase

The agent uses multi-modal understanding to interpret any evidence type:

| Evidence Type | Interpretation Approach |
|---------------|------------------------|
| Text paste | Natural language understanding |
| Image | Vision + OCR |
| Screenshot | Vision + layout analysis |
| URL | Fetch + HTML understanding |
| Spreadsheet | Structure recognition + row interpretation |
| PDF | Document parsing + vision |

The prompt is not:

> "Extract the following fields: artist, venue, date, time"

The prompt is:

> "You are looking at evidence about live music events. What does this appear to tell us about the live music world? What claims can you make? What are you uncertain about?"

## The Comparison Phase

The agent queries existing memory:

```typescript
// Pseudocode for memory comparison
async function compareWithMemory(claims: Claim[]): Promise<ComparisonResult> {
  for (const claim of claims) {
    // Check for existing entities
    const existingEntity = await memory.findSimilar(claim.subject);

    if (existingEntity) {
      // Check for contradiction
      const conflicts = await memory.findConflicts(claim, existingEntity);

      if (conflicts.length > 0) {
        return { status: 'conflicted', conflicts };
      }

      // This corroborates existing knowledge
      claim.corroborates = existingEntity.id;
    } else {
      // This proposes new knowledge
      claim.proposesNew = true;
    }
  }

  return { status: 'ready_to_propose', claims };
}
```

## The Proposal Phase

The agent proposes changes to the world model:

```typescript
interface WorldStateProposal {
  proposalId: string;
  signalId: string;

  // What should change
  createEntities: ProposedEntity[];
  updateEntities: ProposedUpdate[];
  createRelationships: ProposedRelationship[];
  attachEvidence: EvidenceAttachment[];

  // Why
  reasoning: string;
  claims: Claim[];

  // Confidence
  overallConfidence: 'high' | 'medium' | 'low';
  uncertainties: string[];

  // Recommendation
  recommendation: 'auto_accept' | 'human_review' | 'reject';
}
```

## Auto-Acceptance Rules

The agent can auto-accept when:

| Condition | Threshold |
|-----------|-----------|
| All entities already known | venue + artist exist |
| No contradictions | 0 conflicts |
| Clear interpretation | uncertainty count < 2 |
| Source is trusted | source reliability > 0.8 |
| Claims are simple | no complex inferences |

When any condition fails, route to human review.

## Example: High Confidence

```
Signal: Facebook post from Stingray's verified page

Agent interpretation:
- Event announced: Stingray @ The Rigger, 15 May 2026
- The Rigger: Known venue (ID vnue_123), 47 prior events
- Stingray: Known artist (ID arts_456), 3 prior events
- No contradictions found
- Source: Verified artist page (high trust)

Proposal:
- Create event evnt_789
- Link to vnue_123, arts_456
- Attach signal as primary evidence

Recommendation: AUTO_ACCEPT
```

## Example: Needs Review

```
Signal: Screenshot of poster, partially obscured

Agent interpretation:
- Appears to be event announcement
- Venue text: "The Ri[obscured]" - could be The Rigger or The Rising Sun
- Date text: "15th" - month unclear
- Artist: "Sting[obscured]" - could be Stingray or different artist

Proposal:
- Possible event, uncertain
- Venue: 60% The Rigger, 30% The Rising Sun
- Artist: 70% Stingray, 30% unknown

Recommendation: HUMAN_REVIEW
Reason: Multiple ambiguities, cannot confidently match entities
```

## Example: Contradiction

```
Signal: Venue website calendar

Agent interpretation:
- Event: Stingray @ The Rigger, 15 May 2026
- BUT: Existing claim says Stingray @ The Underground, 15 May 2026

Conflict detected:
- Signal 1 (Facebook): Claims The Rigger
- Signal 2 (Venue web): Claims The Rigger
- Signal 3 (Artist web): Claims The Underground

Proposal: CANNOT_AUTO_ACCEPT
Reason: Contradiction between sources
Required: Human resolution

Challenge question to human:
"We have conflicting information about where Stingray are playing on 15 May.
Facebook and The Rigger's website say The Rigger.
Stingray's website says The Underground.
Which is correct?"
```

## The Loop Continues

After acceptance or rejection, the loop doesn't end:

1. Memory is updated with new claims/evidence
2. Source reliability is adjusted (up if accurate, down if corrected)
3. Related claims may be strengthened or weakened
4. Future signals benefit from updated memory

This is learning by reasoning, not training by gradient descent.

## Related

- [[bndy-brain-concept]] - The overall model
- [[signal-to-claim-model]] - How signals become claims
- [[human-challenge-loop]] - How humans correct the agent

