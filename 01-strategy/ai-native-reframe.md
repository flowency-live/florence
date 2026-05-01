# AI-Native Reframe

## What "AI-Native" Means for bndy

AI-native is not "we added ChatGPT to our app."

AI-native means the entire system is designed around the assumption that:

1. **AI can read everything** - All data is structured for machine consumption
2. **AI can reason about context** - Relationships between entities matter
3. **AI can take action** - Agents can ingest, classify, recommend, communicate
4. **AI improves continuously** - Every interaction generates learning signal

## Traditional vs AI-Native

| Traditional App | AI-Native Platform |
|-----------------|-------------------|
| Users manually enter events | AI ingests events from 50+ sources |
| Categories are static taxonomies | Genres emerge from artist analysis |
| Recommendations = collaborative filtering | Recommendations = contextual reasoning |
| Search = keyword matching | Search = natural language understanding |
| Moderation = human review | Moderation = AI triage + human escalation |

## Design Principles

### 1. Structured data over free text

Every entity (venue, artist, event) has a canonical schema. Free text exists for humans; structured data exists for AI.

### 2. Graph over tables

Relationships matter. "This artist played at this venue" is more valuable than two isolated records.

### 3. Confidence scores over binary truth

Is this the same venue? Maybe 85% confidence. Let AI act on probability, escalate uncertainty.

### 4. Agents over features

Instead of building a "duplicate detection feature", build a "venue identity agent" that continuously resolves duplicates.

### 5. Context windows over databases

Design for AI that can hold entire conversations in memory. Don't force users through modal dialogs - let them talk.

## Implementation Implications

- **Data model**: Entity-relationship graph with confidence scores
- **Ingestion**: Multi-source pipeline with AI classification
- **API design**: Rich context in responses, not just IDs
- **Frontend**: Conversational interfaces alongside traditional UI
- **Ops**: Agents running continuously, not just request-response

## Related

- [[bndy-strategy]]
- [[04-architecture/data-model|Data Model]]
