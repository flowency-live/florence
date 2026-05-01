# ADR-002: DynamoDB as Primary Data Store

## Status

**Status:** Accepted
**Date:** 01/05/2026
**Deciders:** Jason

## Context

bndy needs a primary data store for:
- Venues, artists, events, users
- High read volume, moderate write volume
- Geographic queries (events near me)
- Flexible schema (entities evolve)
- Low operational overhead

Options considered:
1. DynamoDB
2. PostgreSQL (RDS)
3. MongoDB Atlas
4. PlanetScale (MySQL)

## Decision

We decided to use DynamoDB with single-table design because:

> In the context of bndy's serverless-first architecture,
> facing the need for a scalable, low-ops data store,
> we decided to use DynamoDB with single-table design,
> to achieve seamless Lambda integration, auto-scaling, and pay-per-request pricing,
> accepting increased query complexity and the need for GSIs.

## Consequences

### Positive

- **Serverless-native**: Direct Lambda integration, no connection pooling issues
- **Auto-scaling**: Handles traffic spikes without intervention
- **Pay-per-request**: Low cost at low volume, scales linearly
- **Low ops**: No patching, backups handled, HA built-in
- **Fast**: Single-digit millisecond reads
- **Flexible schema**: No migrations for new fields

### Negative

- **Query complexity**: Single-table design requires careful modelling
- **No JOINs**: Denormalisation required, data duplication
- **GSI costs**: Global secondary indexes add cost
- **Geo queries**: Need OpenSearch or custom solution for radius search
- **Vendor lock-in**: AWS-specific, migration would be significant

### Neutral

- Team needs to learn DynamoDB access patterns
- Will need OpenSearch for full-text search anyway

## Access Patterns

Primary patterns driving the design:

| Pattern | Access | GSI Needed |
|---------|--------|------------|
| Get venue by ID | PK lookup | No |
| Get events by city + date | Query | GSI1 |
| Get events by venue | Query | No (SK range) |
| Get artist by ID | PK lookup | No |
| Get events by artist | Query | GSI2 |
| Search venues by name | OpenSearch | N/A |

## Alternatives Considered

### Option 1: PostgreSQL (RDS)

**Pros:**
- Familiar SQL
- Rich query capabilities
- Strong consistency
- PostGIS for geo queries

**Cons:**
- Connection pooling complexity with Lambda
- Operational overhead (patching, sizing)
- Fixed capacity pricing
- Cold start impact

**Why rejected:** Lambda cold starts and connection limits make this painful.

### Option 2: MongoDB Atlas

**Pros:**
- Flexible schema
- Good geo queries
- Familiar to many

**Cons:**
- Another vendor (not AWS-native)
- Connection pooling still an issue
- Cost less predictable

**Why rejected:** Adding another vendor increases complexity.

### Option 3: PlanetScale

**Pros:**
- Serverless MySQL
- No connection limits
- Good DX

**Cons:**
- Another vendor
- Less mature than DynamoDB
- Geo queries not native

**Why rejected:** DynamoDB better integrated with AWS stack.

## Related

- [[04-architecture/aws-stack|AWS Stack]]
- [[04-architecture/data-model|Data Model]]
