# bndy v1 Platform Overview

The existing production platform - what we have today.

## What bndy v1 Is

Multi-sided platform connecting the UK grassroots music ecosystem:

- **For Artists/Bands**: Calendar management, setlist coordination, member availability, gig booking, venue CRM
- **For Venues**: Event promotion, performer discovery, audience analytics
- **For Audiences**: Discover local gigs via map-based discovery (Frontstage)

## The Problem Solved

- 125 grassroots UK venues lost annually due to invisibility
- Events scattered across Facebook, WhatsApp, emails, spreadsheets
- Artists can't easily find gigs; venues can't easily find performers
- No unified discovery mechanism for grassroots music

## Current Status

| Status | Details |
|--------|---------|
| Phase | Beta - Invite-only |
| User Limit | 10 bands max |
| Primary Focus | Band/collective management (Backstage) |
| Secondary Focus | Public event discovery (Frontstage) |

## Platform Metrics

| Metric | Value |
|--------|-------|
| Venues | **611** |
| Artists | **597** |
| Events | **1,041** |
| Lambda Functions | 16 |
| API Routes | 125 |
| DynamoDB Tables | 19 |

*Counts as of 01/05/2026 from DynamoDB*

## Three Frontend Applications

| App | Domain | Framework | Purpose |
|-----|--------|-----------|---------|
| **bndy-backstage** | www.bndy.co.uk | React 18 + Vite 5 | Artist CRM, band management |
| **bndy-frontstage** | live.bndy.co.uk | Next.js 15.1.7 | Public event discovery |
| **bndy-centrestage** | — | — | Admin dashboard (legacy) |

## Core Strengths

- Serverless architecture (100% AWS)
- TypeScript strict mode
- Comprehensive documentation (118 files)
- Sophisticated venue deduplication (4-level matching)
- Multi-band membership model

## Key Challenges

- Input validation gaps (no Zod on Lambda handlers)
- Service layer bypass (204 direct API calls)
- Conflict checking stubbed
- Test coverage ~18%

## Documentation Location

Primary docs in `C:\VSProjects\bndy All Platform Docs`:
- `BNDY_PLATFORM_BIBLE.md` - Definitive tech reference
- `architecture/infrastructure.md` - AWS audit
- `reference/api-routes.md` - 125 routes
- `reference/lambda-functions.md` - 16 functions

## Related

- [[architecture]] - AWS infrastructure details
- [[data-models]] - TypeScript interfaces
- [[api-routes]] - All 125 endpoints
- [[technical-debt]] - Known issues
- [[migration-notes]] - v2 planning
