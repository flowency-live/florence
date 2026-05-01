# bndy v1 Data Models

TypeScript interfaces for all v1 entities.

## Event

```typescript
interface Event {
  id: string;                           // UUID
  name: string;                         // Event title
  title?: string;                       // Alternative title field
  type: 'gig' | 'unavailable' | 'rehearsal' | 'other';
  date: string;                         // YYYY-MM-DD
  startTime: string;                    // HH:mm
  endTime?: string;                     // HH:mm
  isAllDay?: boolean;

  // Relationships
  venueId: string;
  venueName: string;
  venueCity?: string;
  artistIds: string[];                  // Multiple artists supported
  artistId?: string;                    // Legacy single artist
  artistName?: string;                  // Display name
  ownerUserId?: string;                 // For personal events (XOR with artistId)
  membershipId?: string;                // Artist membership context

  // Location
  location: { lat: number; lng: number };
  geoLat?: number;                      // Legacy
  geoLng?: number;                      // Legacy
  geohash4?: string;                    // Rough location (spatial index)
  geohash6?: string;                    // Precise location (spatial index)
  postcode?: string;

  // Ticketing
  price?: string | null;
  ticketed: boolean;
  ticketinformation?: string;
  ticketUrl?: string;
  eventUrl?: string;

  // Metadata
  description?: string;
  imageUrl?: string;
  source: 'bndy.live' | 'user' | 'bndy.core' | 'backstage_wizard';
  status: 'pending' | 'approved' | 'rejected';
  isPublic?: boolean;
  isOpenMic?: boolean;
  verifiedByArtist?: boolean;
  hasCustomTitle?: boolean;
  notes?: string | null;

  createdAt: string;
  updatedAt: string;
}
```

## Artist

```typescript
interface Artist {
  id: string;                           // UUID
  name: string;
  nameVariants?: string[];              // Alternate names for matching

  // Classification
  artist_type?: 'band' | 'solo' | 'duo' | 'trio' | 'group' | 'dj' | 'collective';
  artistType?: string;                  // Compatibility alias
  genres?: string[];                    // ["Rock", "Jazz", etc.]
  acoustic?: boolean;
  actType?: ('originals' | 'covers' | 'tribute')[];

  // Profile
  bio?: string;                         // NOT 'description'
  location?: string;                    // Region/city
  profileImageUrl?: string;
  displayColour?: string;               // Fallback color when no image

  // Social
  socialMediaUrls?: SocialMediaURL[];
  facebookUrl?: string;                 // Legacy
  instagramUrl?: string;                // Legacy
  websiteUrl?: string;                  // Legacy
  youtubeUrl?: string;                  // Legacy
  spotifyUrl?: string;                  // Legacy
  twitterUrl?: string;                  // Legacy

  // Ownership
  claimedByUserId?: string | null;      // Owner user ID
  isVerified?: boolean;
  publishAvailability?: boolean;

  createdAt?: string;
  updatedAt?: string;
}
```

## Venue

```typescript
interface Venue {
  id: string;                           // UUID
  name: string;
  nameVariants?: string[];              // Alternate names for matching

  // Location
  location: { lat: number; lng: number };
  latitude?: number;                    // Legacy
  longitude?: number;                   // Legacy
  address: string;
  city?: string;
  postcode?: string;
  googlePlaceId?: string;               // Google Places integration

  // Contact
  phone?: string;
  email?: string;
  website?: string;
  socialMediaUrls?: SocialMediaURL[];

  // Profile
  description?: string;
  imageUrl?: string;
  profileImageUrl?: string | null;
  facilities?: string[];                // ["WiFi", "Food", "Parking", etc.]

  // Defaults for events
  standardStartTime?: string;
  standardEndTime?: string;
  standardTicketed?: boolean;
  standardTicketUrl?: string;
  standardTicketInformation?: string;

  // Status
  validated: boolean;                   // Admin-verified

  createdAt: string;
  updatedAt: string;
}
```

## User

```typescript
interface User {
  userId: string;                       // UUID (from auth)
  id?: string;                          // Alias
  email?: string;
  phone?: string;
  displayName?: string;
  photoURL?: string | null;
  profileImageUrl?: string | null;
  postcode?: string;                    // UK postcode for location
  instruments?: string[];
  bio?: string | null;

  created_at: string;
  updated_at: string;
}
```

## ArtistMembership

```typescript
interface ArtistMembership {
  membership_id: string;                // UUID
  artist_id: string;
  user_id: string;

  // Role & Status
  role: 'owner' | 'admin' | 'member';
  status: 'active' | 'pending' | 'inactive';
  membership_type: 'performer' | 'manager' | 'crew';

  // Profile overrides (NULL = inherit from user)
  display_name?: string | null;
  bio?: string | null;
  instrument?: string | null;
  avatar_url?: string | null;
  icon?: string;                        // Font Awesome class
  color?: string;                       // Hex color for UI

  // Permissions
  permissions: string[];                // invite_members, manage_events, edit_band, remove_members, manage_songs

  // Tracking
  invited_by_user_id?: string;
  invited_at?: string;
  joined_at?: string;
  created_at: string;
  updated_at: string;
}
```

## Song & ArtistSong

```typescript
interface Song {
  id: string;
  title: string;
  originalArtist?: string;              // Who wrote it
  duration?: number;                    // Seconds
  key?: string;
  bpm?: number;
  genre?: string;
  spotifyId?: string;
  spotifyUrl?: string;
}

interface ArtistSong {
  artistSongId: string;
  artistId: string;
  songId: string;

  // Pipeline
  status: 'suggestion' | 'voting' | 'in_progress' | 'ready' | 'rejected';

  // Voting
  votes: { userId: string; vote: 'yes' | 'no'; timestamp: string }[];

  // RAG Status (per member)
  ragStatus: { userId: string; status: 'red' | 'amber' | 'green' }[];

  // Comments
  comments: { userId: string; text: string; timestamp: string }[];

  created_at: string;
  updated_at: string;
}
```

## Setlist

```typescript
interface Setlist {
  setlistId: string;
  artistId: string;
  name: string;

  sets: {
    setNumber: number;
    songs: {
      songId: string;
      order: number;
      notes?: string;
    }[];
  }[];

  created_at: string;
  updated_at: string;
}
```

## Invite

```typescript
interface Invite {
  token: string;                        // Primary key
  artistId: string;

  // Target
  email?: string;
  phone?: string;

  // Tracking
  status: 'pending' | 'accepted' | 'rejected' | 'expired';
  invited_by_user_id: string;
  expires_at: string;                   // 7-day TTL

  created_at: string;
}
```

## Supporting Types

```typescript
type SocialPlatform = 'website' | 'spotify' | 'facebook' | 'instagram' | 'youtube' | 'x';

interface SocialMediaURL {
  platform: SocialPlatform;
  url: string;
}

interface EventWithDistance extends Event {
  distanceMiles: number | null;
}
```

## Domain Terminology

### Artist Types

| Type | Description |
|------|-------------|
| band | 3+ members performing together |
| solo | Single performer |
| duo | Two performers |
| group | Ensemble (jazz, choir) |
| dj | Electronic/mixing performer |
| collective | Loose group of musicians |

### Event Types

| Type | Description |
|------|-------------|
| gig | Public performance (artistId + venueId) |
| unavailable | Personal unavailability (ownerUserId only) |
| rehearsal | Band practice |
| other | Custom event types |

### Membership Roles

| Role | Description |
|------|-------------|
| owner | Full control, created the band |
| admin | Can invite, manage events, edit band |
| member | Basic access, mark availability |

### Song RAG Status

| Status | Description |
|--------|-------------|
| Red | Not ready, concerns flagged |
| Amber | Needs practice |
| Green | Performance ready |

### Pipeline Statuses

| Status | Description |
|--------|-------------|
| suggestion | Proposed song |
| voting | Members voting yes/no |
| in_progress | Approved, being learned |
| ready | Performance ready (in playbook) |
| rejected | Vetoed/declined |

## Related

- [[overview]]
- [[architecture]]
- [[05-entities/venue-model|v2 Venue Model]]
