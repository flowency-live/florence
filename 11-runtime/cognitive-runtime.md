# Cognitive Runtime

## The Core Shift

Signals are not "processed".

Signals continuously contribute to evolving world understanding.

```
OLD: request → process → done
NEW: evidence → interpret → remember → evolve
```

## What This Means

### Not a Pipeline

A pipeline processes data and finishes:

```
Input → Extract → Transform → Load → Done
```

The cognitive runtime never finishes:

```
Evidence arrives
↓
Interpret
↓
Remember
↓
New evidence arrives
↓
Re-interpret in context
↓
Understanding evolves
```

### Signals Are Long-Lived

A signal doesn't get "processed and archived".

A signal remains in the system:
- Its interpretations can be superseded
- New evidence can cause re-interpretation
- Its contribution to knowledge is tracked forever

### Interpretations Are Versioned

Understanding evolves:

```
May 1: Signal interpreted
  → "The Rigger" uncertain

May 5: Venue evidence arrives
  → Re-interpret: "The Rigger" = vnue_123

May 10: Event confirmed
  → Re-interpret: confidence strong
```

Each interpretation is a snapshot of understanding at that moment.

## The Cognitive Loop

```
┌─────────────────────────────────────────────────────────────┐
│                    COGNITIVE RUNTIME                         │
│                                                              │
│  ┌─────────┐    ┌─────────────┐    ┌─────────────┐          │
│  │ SIGNAL  │───▶│INTERPRETATION│───▶│   CLAIMS    │          │
│  └─────────┘    └─────────────┘    └─────────────┘          │
│       │               │                  │                   │
│       │               │                  │                   │
│       ▼               ▼                  ▼                   │
│  ┌─────────────────────────────────────────────────┐        │
│  │                   MEMORY                         │        │
│  │  • Signals persist                               │        │
│  │  • Interpretations versioned                     │        │
│  │  • Claims tracked                                │        │
│  │  • Evidence packs grow                           │        │
│  └─────────────────────────────────────────────────┘        │
│       │                                                      │
│       │  New evidence or context change                     │
│       ▼                                                      │
│  ┌─────────────────────────────────────────────────┐        │
│  │              RE-INTERPRETATION                    │        │
│  │  • Previous interpretation superseded             │        │
│  │  • New interpretation with updated context        │        │
│  │  • Claims refined                                 │        │
│  └─────────────────────────────────────────────────┘        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## Key Differences from Traditional Systems

| Traditional | Cognitive Runtime |
|-------------|-------------------|
| Process data | Interpret evidence |
| Store records | Remember understanding |
| Update in place | Version interpretations |
| Delete old | Supersede with history |
| Calculate confidence | Assess strength with reasoning |
| Match by rules | Reason about meaning |
| Done when processed | Evolves continuously |

## Evidence Accumulation

Knowledge grows through accumulation:

```
Day 1: Single Facebook post
  → Weak evidence
  → Low confidence interpretation

Day 3: Poster image matches
  → Moderate evidence
  → Corroboration strengthens

Day 5: Venue website confirms
  → Strong evidence
  → High confidence

Day 7: Event happens
  → Historical fact
  → Can inform future predictions
```

The runtime tracks this entire evolution.

## Re-Interpretation Triggers

The runtime may re-interpret when:

| Trigger | What Happens |
|---------|--------------|
| New corroborating signal | Add to evidence pack, strengthen |
| Contradicting signal | Flag conflict, request review |
| Venue database update | Re-match venue references |
| Artist database update | Re-match artist references |
| Model improvement | Re-interpret with better model |
| Human challenge | Re-interpret with correction context |
| Time passage | Event becomes historical fact |

## Memory Structure

```
Memory
├── Signals (immutable raw evidence)
│   └── Never deleted, always accessible
│
├── Interpretations (versioned understanding)
│   ├── v1: Initial interpretation
│   ├── v2: After new evidence
│   └── v3: After model improvement
│
├── Claims (proposed world-state)
│   ├── Linked to interpretation
│   └── Can be accepted, challenged, superseded
│
├── Evidence Packs (corroboration)
│   ├── Groups related signals
│   └── Tracks strength growth
│
└── Canonical Entities (published knowledge)
    ├── Derived from strong evidence
    └── Linked to source evidence
```

## The Brain vs The Database

Traditional:
```
Database stores facts.
Facts are true or false.
Updates replace old values.
```

Cognitive Runtime:
```
Memory stores understanding.
Understanding has confidence.
New understanding supersedes, doesn't replace.
History is preserved.
```

## Practical Implications

### For Storage

- Never delete signals
- Version all interpretations
- Track supersession chains
- Preserve reasoning

### For Queries

- "What do we know about X?" → Current understanding
- "Why do we think X?" → Reasoning chain
- "What evidence supports X?" → Source signals
- "How has our understanding changed?" → Version history

### For UI

- Show confidence with explanation
- Show source evidence
- Show interpretation history
- Allow challenge with context

### For Costs

- Track per-interpretation costs
- Budget for re-interpretation
- Monitor model efficiency
- Alert on spend

## Phase 1 Simplifications

For Phase 1, the runtime is simplified:

| Full Runtime | Phase 1 |
|--------------|---------|
| Automatic re-interpretation | Manual only |
| Auto-publish strong evidence | All goes to review |
| Evidence pack auto-merge | Manual linking |
| Model improvement triggers | Not implemented |

The architecture supports full runtime; Phase 1 uses a subset.

## Why This Matters

### For Accuracy

Traditional systems are only as accurate as their last update.

Cognitive systems accumulate evidence over time. More signals = more confidence.

### For Explainability

Traditional: "The system says X."

Cognitive: "We believe X because of signals A, B, C with this reasoning..."

### For Correction

Traditional: Edit the record.

Cognitive: Challenge with context. Old understanding preserved. New understanding linked to correction.

### For Trust

Traditional: Trust the data or don't.

Cognitive: See exactly why we believe something and how strongly.

## The Ultimate Goal

A user should be able to ask:

> "Why do you think Stingray is playing at The Rigger on May 15th?"

And the runtime should answer:

> "We have three independent sources:
> 1. Facebook post from the band (May 1)
> 2. Poster image uploaded (May 3)
> 3. Gigantic ticket listing (May 4)
>
> All three agree on the event, date, and venue.
> The venue matched to our existing record for The Rigger (vnue_123) based on name and location.
> The artist matched to Stingray (arts_456) based on the same Facebook page we've seen before.
>
> Corroboration strength: strong.
> We're confident this event exists."

That's the cognitive runtime.

## Related

- [[../10-brain/bndy-brain-concept|Brain Concept]] - The overall model
- [[../10-brain/memory-and-reasoning-model|Memory and Reasoning]] - How bndy remembers
- [[../05-entities/interpretation-model|Interpretation Model]] - Versioned understanding
- [[../05-entities/evidence-pack-model|Evidence Pack Model]] - Corroboration

