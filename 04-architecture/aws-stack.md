# AWS Stack

bndy v2 infrastructure on AWS.

## Design Principles

> Start serverless and event-driven. Add specialist services only when the use case and volume justify them.

1. **Serverless-first** - No servers to manage, scale-to-zero costs
2. **Event-driven** - Loosely coupled via events, not direct calls
3. **CDK-native** - Infrastructure as code, no ClickOps
4. **Cost-aware** - Start cheap, scale when needed
5. **AI-selective** - Use LLMs for ambiguity, not for CRUD

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         CloudFront                               │
│                    (CDN + edge caching)                          │
└─────────────────────┬───────────────────────────────────────────┘
                      │
          ┌───────────┴───────────┐
          │                       │
          ▼                       ▼
┌─────────────────┐     ┌─────────────────┐
│   S3 (Static)   │     │   API Gateway   │
│   Next.js SSG   │     │   HTTP API v2   │
└─────────────────┘     └────────┬────────┘
                                 │
                                 ▼
                        ┌─────────────────┐
                        │     Lambda      │
                        │   (handlers)    │
                        └────────┬────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
              ▼                  ▼                  ▼
     ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
     │  DynamoDB   │    │ EventBridge │    │     SQS     │
     │   (data)    │    │  (events)   │    │  (queues)   │
     └─────────────┘    └─────────────┘    └─────────────┘
```

## Core Application Layer

### Frontend

| Option | Use Case |
|--------|----------|
| S3 + CloudFront | Static-first delivery (preferred) |
| Amplify Hosting | If CI/CD and branch previews matter more than lowest cost |

Notes:
- Static-first delivery is efficient and cheap
- Avoid heavy server-side rendering unless strong SEO need
- Use static generation or incremental regeneration for event pages

### Authentication

| Option | Trade-off |
|--------|-----------|
| **Cognito** | Cost-effective at scale, less pleasant DX |
| Firebase Auth | If already stable, don't migrate prematurely |
| Clerk/Auth0 | Speed and DX over cost control |

Authentication is not the strategic moat. Avoid burning time here unless it blocks the product.

### API Layer

| Service | Purpose |
|---------|---------|
| API Gateway HTTP API | REST endpoints |
| Lambda | Request handlers (Node.js) |
| Lambda Powertools | Logging, tracing, idempotency |
| IAM | Least privilege |

Avoid ECS/Fargate until there is a clear workload that needs containers.
Avoid AppSync unless GraphQL provides genuine product leverage.

## Data Layer

### Primary Data Store: DynamoDB

Use DynamoDB for:
- Events, artists, venues, users
- Claims, source records, verification records
- Contribution records, trust scores

Design principles:
- Single-table or carefully grouped table design
- Predictable access patterns
- GSIs for location/time/entity lookups
- TTL for temporary extraction jobs
- Streams for event-driven processing

Do not move to a graph database too early. Model graph relationships explicitly in DynamoDB first.

### Search and Discovery

| Phase | Solution |
|-------|----------|
| **Start** | DynamoDB queries for simple map/time filters |
| **Next** | Typesense or Meilisearch for search UX |
| **Later** | OpenSearch Serverless when volume justifies |

### Geospatial Search

| Phase | Solution |
|-------|----------|
| **Start** | Geohash/S2 + DynamoDB queries |
| **Later** | OpenSearch geo queries or PostGIS if genuinely complex |

Avoid expensive map tile usage through caching, clustering and sensible viewport queries.

## AI/ML Layer

### OCR and Image Extraction

| Option | Use Case |
|--------|----------|
| Tesseract | Simple OCR on Lambda (cheaper) |
| Amazon Textract | Document extraction (more powerful) |
| Bedrock multimodal | LLM-based extraction and reasoning |

Recommended pipeline:
1. Upload image to S3
2. Run OCR extraction
3. Pass extracted text + image context to LLM
4. Produce structured candidate event JSON
5. Ask human to confirm

### LLM Layer (Bedrock)

Use cases:
- Event extraction from text/posters
- Entity matching suggestions
- Duplicate detection explanation
- Genre/scene classification
- Trust/confidence reasoning
- Natural language search

**Economical principles:**
- Use cheap deterministic code first where possible
- Use LLMs for ambiguity, extraction and reasoning
- Do NOT use LLMs for simple lookups, geocoding, CRUD or deterministic validation
- Use smaller models for classification, larger for complex reasoning
- Store structured outputs to avoid reprocessing

### Vector Search (Defer)

Options when needed:
- OpenSearch vector engine
- Aurora PostgreSQL with pgvector
- Qdrant (economical if self-hosted)

Start with embeddings stored alongside entities.

## Graph Layer (Defer)

### Phase 1: DynamoDB Graph Overlay

Store relationships as items:
```
PK = ENTITY#artist_123
SK = REL#played_at#venue_456
```

Pros: cheap, simple, uses current stack, good enough early
Cons: complex graph traversals are harder

### Phase 2: Dedicated Graph DB (When Justified)

Options:
- Amazon Neptune (AWS-native, cost/ops complexity)
- Neo4j/Memgraph (strong DX, extra platform)

Only add when product features genuinely need multi-hop traversal or graph algorithms.

## Event-Driven Processing

| Service | Purpose |
|---------|---------|
| EventBridge | Domain events |
| SQS | Queues and retries |
| Lambda | Processors |
| Step Functions | Longer workflows (only when needed) |
| DynamoDB Streams | Change-driven enrichment |

Example events:
- `PosterUploaded`
- `EventExtracted`
- `CandidateEntityMatched`
- `EventVerified`
- `VenueClaimed`
- `DuplicateDetected`
- `TrustScoreChanged`

## File and Image Storage

| Bucket/Prefix | Purpose |
|---------------|---------|
| `raw-uploads/` | Original uploads |
| `processed/` | Processed images |
| `thumbnails/` | Reduced transfer costs |
| `evidence/` | Private extraction artefacts |
| `public/` | Public assets via CloudFront |

Use lifecycle policies to move old raw artefacts to cheaper storage.

## Analytics (Start Simple)

| Phase | Solution |
|-------|----------|
| **Start** | DynamoDB Streams → S3 → Athena |
| **Later** | Glue Data Catalog, QuickSight |
| **Defer** | Redshift until serious analytical volume |

Key metrics to track:
- Extraction success rate
- Extraction cost per event
- Event verification rate
- Duplicate detection rate
- Source reliability
- Human correction rate
- LLM token spend

## Observability

| Service | Purpose |
|---------|---------|
| CloudWatch | Logs, metrics, alarms |
| Lambda Powertools | Structured logging |
| X-Ray | Tracing where needed |
| Cost Anomaly Detection | Catch runaway spend |

**Critical:** Track AI cost per successfully verified event. Without this, LLM spend can quietly become wasteful.

## Economical Stack Summary

### Start With
- Frontend: Next.js PWA on S3 + CloudFront
- API: API Gateway HTTP API + Lambda
- Database: DynamoDB
- Storage: S3 + CloudFront
- Async: SQS + Lambda
- Events: EventBridge for domain events
- AI: Bedrock selectively
- Search: DynamoDB/geohash first
- Analytics: S3 + Athena
- Monitoring: CloudWatch + structured logs

### Defer Until Justified
- Amazon Neptune
- Redshift
- OpenSearch Serverless
- ECS/Fargate services
- Heavy BI tooling
- Always-on graph/vector infrastructure

## Related

- [[data-model]]
- [[ingestion-pipeline]]
- [[06-decisions/ADR-002-dynamodb-first|ADR-002: DynamoDB First]]
- [[08-v1-platform/architecture|v1 Architecture]] (for migration context)
