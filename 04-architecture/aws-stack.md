# AWS Stack

bndy infrastructure on AWS.

## Design Principles

1. **Serverless-first** - No servers to manage, scale-to-zero costs
2. **Event-driven** - Loosely coupled via events, not direct calls
3. **CDK-native** - Infrastructure as code, no ClickOps
4. **Cost-aware** - Start cheap, scale when needed

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
│   Next.js SSG   │     │    REST API     │
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

## Components

### Frontend

| Service | Purpose | Notes |
|---------|---------|-------|
| CloudFront | CDN, edge caching | Global distribution |
| S3 | Static assets, SSG pages | Next.js export |
| Route 53 | DNS | bndy.live domain |

### API

| Service | Purpose | Notes |
|---------|---------|-------|
| API Gateway | REST API, auth | Rate limiting, CORS |
| Lambda | Request handlers | Node.js, per-route functions |
| Cognito | User auth | JWT tokens, social login |

### Data

| Service | Purpose | Notes |
|---------|---------|-------|
| DynamoDB | Primary data store | Single-table design |
| S3 | Large objects | Images, documents |
| OpenSearch | Full-text search | Event/venue search |

### Events & Async

| Service | Purpose | Notes |
|---------|---------|-------|
| EventBridge | Event bus | Decoupled communication |
| SQS | Work queues | Ingestion, processing |
| Step Functions | Workflows | Multi-step processes |

### Ingestion

| Service | Purpose | Notes |
|---------|---------|-------|
| EventBridge Scheduler | Cron triggers | Daily/hourly ingestion |
| Lambda | Scrapers, parsers | Per-source functions |
| S3 | Raw data archive | Source snapshots |

### AI/ML

| Service | Purpose | Notes |
|---------|---------|-------|
| Bedrock | LLM inference | Claude for reasoning |
| Lambda | AI orchestration | Prompt management |

## Environments

| Env | Purpose | Notes |
|-----|---------|-------|
| dev | Local development | LocalStack or real AWS |
| staging | Pre-production | Full stack, test data |
| prod | Production | Real users, real data |

## Cost Estimates (Initial)

| Service | Monthly | Notes |
|---------|---------|-------|
| Lambda | ~$5 | Low traffic initially |
| DynamoDB | ~$10 | On-demand pricing |
| API Gateway | ~$5 | Per-request pricing |
| S3 + CloudFront | ~$5 | Minimal storage |
| **Total** | **~$25/month** | Scale-to-zero |

## Related

- [[data-model]]
- [[ingestion-pipeline]]
- [[06-decisions/ADR-002-dynamodb-first|ADR-002: DynamoDB First]]
