# Signal to Claim Model

## The Problem with Extraction

Traditional data pipelines extract **fields**:

```
Input: "Stingray @ The Rigger, Thursday 15th May, 8pm"

Output: {
  artist: "Stingray",
  venue: "The Rigger",
  date: "2026-05-15",
  time: "20:00"
}
```

This looks correct. But it loses something important:

**What does this actually tell us?**

## Claims, Not Fields

Instead of extracting fields, the agent generates **claims**:

```
Signal: "Stingray @ The Rigger, Thursday 15th May, 8pm"

Claims:
1. There appears to be an event on 15 May 2026
2. "Stingray" appears to be performing
3. "The Rigger" appears to be the venue
4. Doors appear to be at 8pm
5. This appears to be a future event (not historical)
6. This appears to be a live music event (not comedy, DJ, etc.)
```

Claims are interpretations. They carry uncertainty. They invite challenge.

## Claim Structure

```typescript
interface Claim {
  claimId: string;
  signalId: string;

  // What we're claiming
  claimType: ClaimType;
  subject: string;
  predicate: string;
  object: string;

  // Interpretation context
  reasoning: string;
  uncertainty: string[];

  // Agent state
  status: 'proposed' | 'accepted' | 'challenged' | 'rejected';
  challengedBy?: string;
  challengeReason?: string;
}

type ClaimType =
  | 'event_exists'
  | 'artist_performs'
  | 'venue_hosts'
  | 'event_on_date'
  | 'event_at_time'
  | 'event_has_price'
  | 'artist_exists'
  | 'venue_exists'
  | 'source_corroborates';
```

## Example: Facebook Paste

**Signal:**
```
STINGRAY
LIVE AT THE RIGGER
THURSDAY 15TH MAY
DOORS 8PM
£5 OTD
Support from The Wanderers
```

**Claims generated:**

| # | Claim Type | Subject | Predicate | Object | Reasoning |
|---|------------|---------|-----------|--------|-----------|
| 1 | event_exists | event_? | exists | true | "LIVE AT" suggests a performance event |
| 2 | artist_performs | Stingray | performs_at | event_? | Headline position in text |
| 3 | artist_performs | The Wanderers | supports_at | event_? | "Support from" indicates opening act |
| 4 | venue_hosts | The Rigger | hosts | event_? | "AT THE RIGGER" is venue reference |
| 5 | event_on_date | event_? | on_date | 2026-05-15 | "THURSDAY 15TH MAY" interpreted as future |
| 6 | event_at_time | event_? | doors_at | 20:00 | "DOORS 8PM" |
| 7 | event_has_price | event_? | price | £5 | "£5 OTD" means on-the-door price |

**Uncertainties flagged:**
- Year inferred (2026, not stated)
- Venue location not stated
- Is "Stingray" a band name or event name?
- Is "The Wanderers" the actual band name?

## Example: Poster Image

**Signal:** [Image of gig poster]

**Claims generated:**

| # | Claim Type | Subject | Predicate | Object | Reasoning |
|---|------------|---------|-----------|--------|-----------|
| 1 | event_exists | event_? | exists | true | Visual structure suggests event promotion |
| 2 | artist_performs | Stingray | performs_at | event_? | Large text, likely headline |
| 3 | venue_hosts | The Rigger | hosts | event_? | Location text visible |
| 4 | event_on_date | event_? | on_date | 2026-05-15 | Date visible on poster |

**Uncertainties flagged:**
- OCR confidence on date text
- Multiple artist names hard to parse
- Venue address not visible

## Example: Spreadsheet

**Signal:** Promoter's gig list (Excel file)

**Claims generated:** (per row)

Row 1:
```
| # | Claim Type | Subject | Predicate | Object | Reasoning |
|---|------------|---------|-----------|--------|-----------|
| 1 | event_exists | event_? | exists | true | Structured row suggests single event |
| 2 | venue_hosts | The Rigger | hosts | event_? | Column A appears to be venue |
| 3 | event_on_date | event_? | on_date | 2026-05-15 | Column B appears to be date |
| 4 | artist_performs | Stingray | performs_at | event_? | Column C appears to be artist |
```

**Spreadsheet-level uncertainties:**
- Column headers not labelled
- Date format ambiguous (UK vs US)
- Some rows appear to be notes, not events

## Claim Quality

Claims can have quality indicators:

| Indicator | Meaning |
|-----------|---------|
| `clear` | Unambiguous interpretation |
| `inferred` | Reasonable interpretation with gaps |
| `uncertain` | Multiple interpretations possible |
| `guessed` | Low confidence interpretation |

These are not confidence scores. They are descriptions of the interpretation process.

## What Claims Enable

### Contradiction detection

If a new signal generates claims that contradict existing claims, the brain notices:

> Claim: Event X is at The Rigger
> Existing: Event X is at The Underground

This triggers a challenge, not an overwrite.

### Corroboration

If multiple signals generate the same claim, that's corroboration:

> Signal 1 (Facebook): Event on 15 May
> Signal 2 (Gigantic): Event on 15 May
> Signal 3 (Venue website): Event on 15 May

Three independent sources strengthen the claim.

### Partial information

A signal might generate some claims but not others:

> Signal: "Stingray at The Rigger next Thursday"

Claims:
- Artist: Stingray (clear)
- Venue: The Rigger (clear)
- Date: 2026-05-15 (inferred from "next Thursday")
- Time: unknown (no claim generated)

The brain knows what it doesn't know.

## Related

- [[bndy-brain-concept]] - The overall model
- [[claim-evidence-graph]] - How claims link to evidence
- [[agentic-intake-loop]] - How the agent generates claims

