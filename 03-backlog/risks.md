# Risks

Known risks and mitigations for bndy.

---

## Data Risks

### R-001: Data source fragility

**Risk:** External sources change their structure, break ingestion.
**Likelihood:** High
**Impact:** Medium
**Mitigation:**
- Monitor ingestion health daily
- Multiple sources per city for redundancy
- Fallback to community submission

### R-002: Duplicate entities

**Risk:** Same venue/artist exists multiple times with different IDs.
**Likelihood:** High
**Impact:** High
**Mitigation:**
- Entity resolution pipeline with confidence scoring
- Community verification for edge cases
- Regular duplicate audits

### R-003: Stale event data

**Risk:** Events displayed after they've happened or been cancelled.
**Likelihood:** Medium
**Impact:** High (trust-destroying)
**Mitigation:**
- Auto-archive past events
- Source refresh on schedule
- User reporting for cancelled events

---

## Product Risks

### R-004: Cold start problem

**Risk:** New cities have no events, users bounce.
**Likelihood:** High
**Impact:** High
**Mitigation:**
- Don't launch in a city until minimum event threshold met
- Partner with local promoters pre-launch
- Clear "coming soon" messaging for empty cities

### R-005: Community builder churn

**Risk:** Venues/promoters stop submitting after initial enthusiasm.
**Likelihood:** Medium
**Impact:** High
**Mitigation:**
- Show clear value (analytics, audience reach)
- Low-friction submission process
- Regular engagement with top contributors

### R-006: Discovery quality

**Risk:** Recommendations are poor, users don't find relevant events.
**Likelihood:** Medium
**Impact:** High
**Mitigation:**
- Start with location + genre (simple, reliable)
- Build personalisation gradually with feedback
- Human curation for featured/editorial

---

## Technical Risks

### R-007: Scale before product-market fit

**Risk:** Over-invest in infrastructure before validating demand.
**Likelihood:** Medium
**Impact:** Medium
**Mitigation:**
- Start with one city, prove the model
- Use serverless, scale-to-zero architecture
- Delay optimisation until needed

### R-008: AI confidence miscalibration

**Risk:** AI entity matching is confident but wrong.
**Likelihood:** Medium
**Impact:** Medium
**Mitigation:**
- Confidence thresholds for auto-merge
- Human review queue for uncertain matches
- Feedback loop from corrections

---

## Business Risks

### R-009: Platform dependency

**Risk:** Key source (e.g., Facebook) blocks scraping.
**Likelihood:** Medium
**Impact:** High
**Mitigation:**
- Diversify sources aggressively
- Build direct relationships with venues
- Community submission as primary path

### R-010: Competitor moves

**Risk:** Ticketmaster or Spotify enters grassroots discovery.
**Likelihood:** Low
**Impact:** High
**Mitigation:**
- Move fast on community relationships
- Depth in grassroots (they won't care about 100-cap venues)
- Build data moat in key cities

---

## Related

- [[now]] - Current sprint
- [[02-product/principles|Product Principles]]
