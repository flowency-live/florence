# bndy brain index

The central navigation hub for the bndy knowledge base.

## Strategy

- [[01-strategy/bndy-strategy|bndy Strategy]]
- [[01-strategy/ai-native-reframe|AI-Native Reframe]]

## Product

- [[02-product/vision|Vision]]
- [[02-product/personas|Personas]]
- [[02-product/principles|Product Principles]]
- [[02-product/feature-catalogue|Feature Catalogue]]
- [[02-product/intelligence-console|Intelligence Console]] - Knowledge growth observatory

## Backlog

- [[03-backlog/now|Now]] - Current sprint
- [[03-backlog/next|Next]] - Upcoming work
- [[03-backlog/later|Later]] - Future ideas
- [[03-backlog/risks|Risks]] - Known risks and mitigations

## Architecture

- [[04-architecture/aws-stack|AWS Stack]]
- [[04-architecture/data-model|Data Model]]
- [[04-architecture/ingestion-pipeline|Ingestion Pipeline]]

## Entities

- [[05-entities/venue-model|Venue]]
- [[05-entities/artist-model|Artist]]
- [[05-entities/event-model|Event]]
- [[05-entities/community-builder-model|Community Builder]]
- [[05-entities/signal-model|Signal]] - Raw evidence input
- [[05-entities/source-record-model|Source Record]] - Provenance tracking
- [[05-entities/relationship-model|Relationship]] - Graph edges

## Decisions

- [[06-decisions/ADR-000-template|ADR Template]]
- [[06-decisions/ADR-001-obsidian-git-brain|ADR-001: Obsidian + Git as local AI brain]]

## Prompts

- [[07-prompts/claude-code-rules|Claude Code Rules]]
- [[07-prompts/cursor-rules|Cursor Rules]]
- [[07-prompts/chatgpt-product-review|ChatGPT Product Review]]

## v1 Platform (Existing)

The current production platform - context for v2 development.

- [[08-v1-platform/overview|Platform Overview]] - What v1 is, current status
- [[08-v1-platform/architecture|Architecture]] - AWS, Lambda, DynamoDB
- [[08-v1-platform/data-models|Data Models]] - TypeScript interfaces
- [[08-v1-platform/api-routes|API Routes]] - All 125 endpoints
- [[08-v1-platform/technical-debt|Technical Debt]] - Known issues
- [[08-v1-platform/migration-notes|Migration Notes]] - v1 → v2 planning
- [[08-v1-platform/v1-data-audit|v1 Data Audit]] - Data quality assessment

## Migration (v1 → v2)

The architectural hinge point - transforming v1 data into canonical v2 entities.

- [[09-migration/v1-to-canonical-plan|Migration Strategy]] - Overall approach
- [[09-migration/migration-execution-plan|Execution Plan]] - Real DynamoDB migration
- [[09-migration/canonical-id-strategy|Canonical ID Strategy]] - ID generation rules
- [[09-migration/v1-source-record-strategy|Source Record Strategy]] - Provenance for v1 data

## Brain (AI-Native Knowledge Model)

The agentic knowledge system - how bndy interprets evidence, generates claims, and learns.

**Evidence in. Knowledge out.**

- [[10-brain/bndy-brain-concept|Brain Concept]] - The core shift from pipelines to agents
- [[10-brain/signal-to-claim-model|Signal to Claim]] - How evidence becomes claims
- [[10-brain/claim-evidence-graph|Claim-Evidence Graph]] - How claims link to evidence
- [[10-brain/agentic-intake-loop|Agentic Intake Loop]] - The agent's reasoning cycle
- [[10-brain/memory-and-reasoning-model|Memory and Reasoning]] - How bndy remembers
- [[10-brain/human-challenge-loop|Human Challenge Loop]] - How humans correct the brain
- [[10-brain/derived-fields-and-field-corruption|Derived Fields and Field Corruption]] - Why contextual knowledge should not be collapsed into brittle fields
