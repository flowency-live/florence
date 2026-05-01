# Community Builder Model

The power users who drive local music scenes.

## Who Are Community Builders?

Community builders are the connectors:
- **Venue owners/managers** - Control the space
- **Promoters** - Book artists, promote shows
- **Scene connectors** - Tastemakers, influencers
- **Collective organisers** - DIY show runners

They're the 1% who create 90% of the events.

## Why They Matter

### Network Effects

One engaged community builder brings:
- Dozens of events per year
- Connections to local artists
- Credibility in their scene
- Word-of-mouth promotion

### Data Quality

Community builders provide:
- Verified venue information
- Accurate event details
- Artist connections
- Scene context

### Growth Leverage

Acquiring community builders is more efficient than acquiring fans:
- One venue = many events = many fans
- Organic promotion through their channels
- Local credibility transfer

## Schema

```typescript
interface CommunityBuilder {
  // Identity (extends User)
  userId: string;                  // Reference to User
  displayName: string;
  slug: string;

  // Role
  builderType: BuilderType;
  roles: BuilderRole[];            // Can have multiple

  // Associations
  claimedVenues: string[];         // Venue IDs
  claimedArtists: string[];        // Artist IDs (if also an artist)
  associatedBrands: string[];      // Promotion brands

  // Profile
  bio?: string;
  location: {
    city: string;
    region?: string;
  };
  genres: string[];                // Scenes they're active in

  // Contact (private)
  contactEmail?: string;
  contactPhone?: string;

  // Links (public)
  website?: string;
  socialLinks?: {
    instagram?: string;
    facebook?: string;
    twitter?: string;
  };

  // Activity Metrics
  eventCount: number;              // Events created/managed
  venueCount: number;              // Venues managed
  audienceReach?: number;          // Estimated follower reach

  // Trust
  verificationLevel: VerificationLevel;
  verifiedAt?: string;
  trustScore: number;              // 0-1, affects moderation queue

  // Permissions
  canSubmitEvents: boolean;
  canEditVenues: boolean;
  canVerifyEntities: boolean;      // Trusted reviewers

  // Timestamps
  joinedAt: string;
  lastActiveAt: string;
}

type BuilderType =
  | 'venue_operator'
  | 'promoter'
  | 'booking_agent'
  | 'collective'
  | 'media'
  | 'artist_manager'
  | 'other';

type BuilderRole =
  | 'venue_owner'
  | 'venue_manager'
  | 'venue_booker'
  | 'promoter'
  | 'pr'
  | 'journalist'
  | 'blogger'
  | 'radio_dj';

type VerificationLevel =
  | 'unverified'                   // Just signed up
  | 'email_verified'               // Confirmed email
  | 'identity_verified'            // Proven venue/role association
  | 'trusted';                     // Track record of quality
```

## Verification Flow

```
┌─────────────┐
│  Sign Up    │
│ (email)     │
└──────┬──────┘
       │
       ▼
┌─────────────┐     ┌─────────────┐
│   Claim     │────►│   Verify    │
│   Venue     │     │   Claim     │
└─────────────┘     └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
       ┌──────────┐ ┌──────────┐ ┌──────────┐
       │  Email   │ │ Document │ │  Manual  │
       │  Domain  │ │  Upload  │ │  Review  │
       └──────────┘ └──────────┘ └──────────┘
              │            │            │
              └────────────┼────────────┘
                           │
                           ▼
                    ┌─────────────┐
                    │  Verified   │
                    │  Claimant   │
                    └──────┬──────┘
                           │
                           ▼ (over time)
                    ┌─────────────┐
                    │   Trusted   │
                    │   Builder   │
                    └─────────────┘
```

### Verification Methods

| Method | How It Works | Trust Level |
|--------|--------------|-------------|
| Email domain | Venue email matches website domain | High |
| Social proof | Tagged by verified venue account | Medium |
| Document | Upload business registration, license | High |
| Manual review | bndy team verifies manually | Highest |

## Privileges by Level

| Action | Unverified | Email | Identity | Trusted |
|--------|------------|-------|----------|---------|
| Submit events | ❌ | Queued | ✅ | ✅ |
| Edit own venues | ❌ | ❌ | ✅ | ✅ |
| See analytics | ❌ | Basic | Full | Full |
| Verify entities | ❌ | ❌ | ❌ | ✅ |
| Skip moderation | ❌ | ❌ | ❌ | ✅ |

## Engagement Strategy

### Acquisition

1. **Direct outreach** - Email venues in target cities
2. **Event detection** - Spot active promoters, invite them
3. **Referral** - Existing builders invite peers

### Activation

1. **Easy claim flow** - Minimal friction to get started
2. **Immediate value** - Show events already tracked
3. **Quick win** - Submit first event, see it live

### Retention

1. **Analytics** - Show them reach, saves, clicks
2. **Discovery** - Surface artists they might want to book
3. **Community** - Connect with other builders

## Related

- [[02-product/personas|Personas]]
- [[venue-model]]
- [[03-backlog/next|BNDY-006: Venue claim flow]]
