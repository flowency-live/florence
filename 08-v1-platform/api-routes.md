# bndy v1 API Routes

All 125 API endpoints in the v1 platform.

## Authentication (12 routes)

```
POST /auth/phone/request-otp
POST /auth/phone/verify-otp
POST /auth/phone/verify-and-onboard
POST /auth/email/request-magic
POST /auth/check-identity
GET  /auth/magic/{token}
GET  /auth/google
GET  /auth/apple
GET  /auth/callback
GET  /auth/landing
POST /auth/logout
GET  /api/me
```

## Artists (7 routes)

```
GET    /api/artists
POST   /api/artists
GET    /api/artists/search?q=&location=&artist_type=
POST   /api/artists/community
GET    /api/artists/{id}
PUT    /api/artists/{id}
DELETE /api/artists/{id}
```

## Artist Events (15 routes)

```
GET    /api/artists/{artistId}/events
POST   /api/artists/{artistId}/events
POST   /api/artists/{artistId}/events/user
POST   /api/artists/{artistId}/events/check-conflicts
POST   /api/artists/{artistId}/public-gigs
GET    /api/artists/{artistId}/events/{id}
PUT    /api/artists/{artistId}/events/{id}
DELETE /api/artists/{artistId}/events/{id}
GET    /api/artists/{artistId}/calendar?startDate=&endDate=
GET    /api/artists/{artistId}/calendar/export/ical
GET    /api/artists/{artistId}/calendar/url
GET    /api/artists/{artistId}/public-events
GET    /api/events
POST   /api/events/batch
POST   /api/events/community
GET    /api/events/public
```

## Artist Songs/Pipeline (11 routes)

```
GET    /api/artists/{artistId}/pipeline
POST   /api/artists/{artistId}/pipeline/suggestions
POST   /api/artists/{artistId}/pipeline/{songId}/vote
PUT    /api/artists/{artistId}/pipeline/{songId}/status
PUT    /api/artists/{artistId}/pipeline/{songId}/comment
POST   /api/artists/{artistId}/pipeline/{songId}/rag
DELETE /api/artists/{artistId}/pipeline/{songId}
GET    /api/artists/{artistId}/playbook
POST   /api/artists/{artistId}/playbook
PUT    /api/artists/{artistId}/playbook/{songId}
DELETE /api/artists/{artistId}/playbook/{songId}
```

## Setlists (6 routes)

```
GET    /api/artists/{artistId}/setlists
POST   /api/artists/{artistId}/setlists
GET    /api/artists/{artistId}/setlists/{setlistId}
PUT    /api/artists/{artistId}/setlists/{setlistId}
DELETE /api/artists/{artistId}/setlists/{setlistId}
POST   /api/artists/{artistId}/setlists/{setlistId}/copy
```

## Venues (9 routes)

```
GET    /api/venues
POST   /api/venues
POST   /api/venues/find-or-create
GET    /api/venues/{id}
PUT    /api/venues/{id}
DELETE /api/venues/{id}
GET    /api/venues/{venueId}/events
```

## Venue CRM (13 routes)

```
GET    /api/artists/{artistId}/crm/venues
POST   /api/artists/{artistId}/crm/venues
PUT    /api/artists/{artistId}/crm/venues/{venueId}
DELETE /api/artists/{artistId}/crm/venues/{venueId}
GET    /api/artists/{artistId}/crm/venues/{venueId}/contacts
POST   /api/artists/{artistId}/crm/venues/{venueId}/contacts
PUT    /api/artists/{artistId}/crm/venues/{venueId}/contacts/{id}
DELETE /api/artists/{artistId}/crm/venues/{venueId}/contacts/{id}
GET    /api/artists/{artistId}/crm/venues/{venueId}/notes
POST   /api/artists/{artistId}/crm/venues/{venueId}/notes
PUT    /api/artists/{artistId}/crm/venues/{venueId}/notes/{id}
DELETE /api/artists/{artistId}/crm/venues/{venueId}/notes/{id}
GET    /api/artists/{artistId}/crm/venues/{venueId}/gigs
```

## Memberships (5 routes)

```
GET    /api/memberships/me
GET    /api/memberships/all
GET    /api/memberships/artist/{artistId}
PUT    /api/memberships/{membershipId}
DELETE /api/memberships/{membershipId}
```

## Invites (5 routes)

```
GET    /api/artists/{artistId}/invites
POST   /api/artists/{artistId}/invites/general
POST   /api/artists/{artistId}/invites/phone
DELETE /api/artists/{artistId}/invites/{token}
POST   /api/invites/{token}/accept
```

## Users (4 routes)

```
GET    /users
GET    /users/profile
PUT    /users/profile
DELETE /users/{userId}
```

## Notifications (5 routes)

```
GET    /api/notifications
PUT    /api/notifications/{id}/read
PUT    /api/notifications/{id}/dismiss
DELETE /api/notifications/{id}
POST   /api/notifications/mark-all-read
```

## Songs (5 routes)

```
GET    /api/songs
POST   /api/songs
GET    /api/songs/{id}
PUT    /api/songs/{id}
DELETE /api/songs/{id}
```

## Ingest/Agent (9 routes)

```
GET    /api/ingest/queue
POST   /api/ingest/queue/{id}/approve
POST   /api/ingest/queue/{id}/reject
POST   /api/ingest/extract-from-url
POST   /api/ingest/extract-from-html
POST   /api/ingest/extract-venues
POST   /api/ingest/enrich-venue-facebook
POST   /api/ingest/update-venue-placeid
POST   /api/ingest/load-poc
```

## Other (3 routes)

```
GET    /api/spotify/search
POST   /uploads/presigned-url
POST   /api/issues
```

## Route Summary by Function

| Lambda Function | Routes | Responsibility |
|-----------------|--------|----------------|
| AuthFunction | 12 | Phone OTP, email magic links, OAuth, sessions |
| ArtistsFunction | 7 | Artist CRUD, search, community |
| ArtistSongsFunction | 11 | Song pipeline, voting, RAG, playbook |
| EventsFunction | 15 | Event CRUD, calendar, iCal, geospatial |
| EventsAgentFunction | 9 | AI event ingestion, venue enrichment |
| VenuesFunction | 9 | Venue CRUD, find-or-create, dedup |
| VenueCRMFunction | 13 | Artist-venue relationships |
| MembershipsFunction | 5 | Band membership management |
| InvitesFunction | 5 | Membership invitations |
| SetlistsFunction | 6 | Setlist CRUD |
| SongsFunction | 5 | Song catalog |
| UsersFunction | 4 | User profiles |
| NotificationsFunction | 5 | In-app notifications |
| IssuesFunction | 1 | Bug reports |
| UploadsFunction | 1 | S3 presigned URLs |
| SpotifyFunction | 1 | Spotify API proxy |

## Related

- [[architecture]]
- [[data-models]]
