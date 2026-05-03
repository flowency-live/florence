# bndy v1 Architecture

100% AWS Serverless infrastructure.

## Infrastructure Overview

| Component | Details |
|-----------|---------|
| **Region** | eu-west-2 (London) |
| **Account** | 771551874768 |
| **API Gateway** | qry0k6pmd0 → api.bndy.co.uk (HTTP API v2) |
| **Lambda Functions** | 16 (Node.js 20.x) |
| **DynamoDB Tables** | 19 (pay-per-request) |
| **Lambda Layer** | bndy-jwt:2 (aws-sdk v2 + jsonwebtoken v9) |
| **S3 Buckets** | bndy-images (uploads), bndy-lambda-deployments |
| **Cognito** | User Pool eu-west-2_LqtkKHs1P |

## Lambda Functions (16)

| Function | Routes | Size | Responsibility |
|----------|--------|------|----------------|
| **AuthFunction** | 12 | 14.5MB | Phone OTP, email magic links, Google/Apple OAuth, sessions |
| **ArtistsFunction** | 7 | 8.9KB | Artist CRUD, search, community creation |
| **ArtistSongsFunction** | 11 | 8.9KB | Song pipeline, voting, RAG status, playbook |
| **EventsFunction** | 15 | 17.9KB | Event CRUD, calendar, iCal export, geospatial |
| **EventsAgentFunction** | 9 | 7.8KB | AI event ingestion, venue enrichment |
| **VenuesFunction** | 9 | 6.5KB | Venue CRUD, find-or-create, deduplication |
| **VenueCRMFunction** | 13 | 4.4KB | Artist-venue relationships, contacts, notes |
| **MembershipsFunction** | 5 | 5.4KB | Band membership management |
| **InvitesFunction** | 5 | 5.6KB | Membership invitations |
| **SetlistsFunction** | 6 | 3.1KB | Setlist CRUD |
| **SongsFunction** | 5 | 3.1KB | Song catalog |
| **UsersFunction** | 4 | 4.0KB | User profiles |
| **NotificationsFunction** | 5 | 3.1KB | In-app notifications |
| **IssuesFunction** | 5 | 4.4KB | Bug reports |
| **UploadsFunction** | 1 | 3.0KB | S3 presigned URLs |
| **SpotifyFunction** | 1 | 2.2KB | Spotify API proxy |

## DynamoDB Tables (19)

| Table | Items | Primary Key | GSIs | Purpose |
|-------|-------|-------------|------|---------|
| bndy-artists | 275 | id | — | Artist profiles |
| bndy-venues | 284 | id | — | Venue locations |
| bndy-events | 8 | id | ownerUserId-date, artistId-date, venueId-date, geohash6-date | Events |
| bndy-users | — | userId | — | User accounts |
| bndy-artist-memberships | 6 | membership_id | artist_id-index, user_id-index | Band memberships |
| bndy-songs | — | id | — | Song catalog |
| bndy-artist-songs | — | artistSongId | artistId-status-index | Song pipeline |
| bndy-setlists | — | setlistId | — | Setlist templates |
| bndy-user-bands | — | userId+artistId | — | Legacy (migrating) |
| bndy-artist-venues | — | artistId+venueId | — | CRM relationships |
| bndy-artist-venue-contacts | — | contactId | — | Venue contacts |
| bndy-venue-notes | — | noteId | — | Private notes |
| bndy-invites | — | token | — | Membership invites (7-day TTL) |
| bndy-magic-tokens | — | token | — | Email magic links (5-min TTL) |
| bndy-otp-codes | — | phoneNumber | — | Phone OTP (15-min TTL) |
| bndy-oauth-states | — | state | — | CSRF protection (10-min TTL) |
| bndy-notifications | — | notificationId | userId-createdAt-index | In-app notifications (30-day TTL) |
| bndy-issues | — | issueId | — | Bug reports |
| bndy-ingest-queue | — | queueId | — | AI event ingestion |

## Authentication

### Session Management

- **Method**: httpOnly JWT cookies
- **Cookie Name**: `bndy_session`
- **Domain**: `.bndy.co.uk` (cross-subdomain)
- **Expiry**: 90 days
- **Secret**: SSM `/bndy/auth/jwt-secret`

### Auth Methods

1. **Phone OTP**: 6-digit SMS via AWS Pinpoint (15-min expiry)
2. **Email Magic Links**: Single-use tokens via AWS SES (5-min expiry)
3. **Google OAuth**: Cognito integration
4. **Apple Sign In**: Cognito integration

## CORS Configuration

```javascript
const ALLOWED_ORIGINS = [
  'https://www.bndy.co.uk',
  'https://backstage.bndy.co.uk',
  'https://live.bndy.co.uk',
  'http://localhost:5173',
  'http://localhost:3000'
];
```

## Frontstage Architecture

### Technology Stack

- **Framework**: Next.js 15.1.7 (App Router)
- **React**: 18.3.1
- **State**: TanStack Query + React Context
- **Maps**: Mapbox GL + Leaflet fallback
- **Styling**: Tailwind CSS + shadcn/ui

### Gig Map Features

- **5 Temporal States**: Spotlight, Soon, Later, Distant, Past
- **Timeline Scrubber**: Real-time focus window (3d/7d/14d)
- **Coordinate Jitter**: Golden-angle spiral prevents pin overlap
- **Geospatial Indexing**: geohash4/geohash6 for clustering

### Gig Submission

- **/dropzone**: Paste text or drag-and-drop gig posters for AI interpretation
- **/chat**: Conversational interface for guided gig submission via AI chat

Both routes use the signals API to create claims about events, venues, and artists.

## Backstage Architecture

### Technology Stack

- **Framework**: React 18 + Vite 5
- **Routing**: Wouter
- **State**: TanStack Query + React Context
- **Forms**: React Hook Form + Zod
- **Styling**: Tailwind CSS + shadcn/ui

### Service Layer (14 services)

All API calls via centralized service classes:
- auth-service, artists-service, memberships-service
- events-service, songs-service, artist-songs-service
- setlists-service, spotify-service, venues-service
- users-service, invites-service, issues-service
- godmode-service, places-service

## Venue Deduplication (4-Level)

| Level | Confidence | Matching Criteria |
|-------|------------|-------------------|
| 1 | 100% | Google Place ID exact match |
| 2 | 90% | Within 50m radius + 80% name similarity |
| 3 | 70% | 85% name similarity + 50% address token overlap |
| 4 | 0% | No match → create new venue |

## Related

- [[overview]]
- [[data-models]]
- [[api-routes]]
