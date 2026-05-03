# Interpretation Model

## Purpose

An Interpretation is a versioned understanding of what a Signal tells us about the live music world.

Signals are raw evidence. Interpretations are what we make of that evidence.

## Why Separate from Signal?

A Signal can have multiple Interpretations over time:

```
Signal: Facebook event paste (received 1 May)
├── Interpretation v1: "The Rigger" - uncertain match
├── Interpretation v2: Matched to vnue_123 after venue evidence arrived
└── Interpretation v3: Re-interpreted with improved model
```

Separating them allows:
- Version history
- Re-interpretation when context changes
- Cost tracking per interpretation
- Challenge without losing original

## Core Principle

```
Signal = raw evidence (immutable)
Interpretation = understanding (versioned)
Claims = proposed world-state changes (derived from interpretation)
```

## TypeScript Interface

```typescript
interface Interpretation {
  interpretationId: string;       // intp_xxxxxxxx
  signalId: string;               // Parent signal
  version: number;                // 1, 2, 3...

  // Deterministic extraction (cheap, fast)
  deterministicExtraction: DeterministicExtraction;

  // LLM interpretation (reasoning)
  llmInterpretation: LLMInterpretation;

  // Cost tracking (REQUIRED)
  sourceCost: SourceCost;

  // Output
  claims: Claim[];
  uncertainties: string[];
  invalidClaimCount?: number;     // Count of invalid claims filtered during generation
  parseError?: string;            // Error message if LLM output parsing failed

  // Lifecycle
  status: InterpretationStatus;
  createdAt: string;
  supersededBy?: string;          // Later interpretation ID
  supersededAt?: string;
}

interface DeterministicExtraction {
  rawText?: string;               // Cleaned text from HTML/PDF
  tables?: Table[];               // Extracted table structures
  dates?: ExtractedDate[];        // Parsed dates
  urls?: string[];                // Found URLs
  ocrText?: string;               // OCR output for images
  errors?: ExtractionError[];     // Errors during extraction (e.g., OCR failed)
  metadata?: {
    title?: string;
    source?: string;
    extractedAt: string;
  };
}

// Reserved for future partial extraction support (not currently used - extraction failures throw)
interface ExtractionError {
  code: string;                   // e.g., "OCR_FAILED"
  message: string;                // Human-readable error
  source?: string;                // Component that failed (e.g., "textract")
}

interface LLMInterpretation {
  modelUsed: string;              // e.g., "eu.anthropic.claude-haiku-4-5-20251001-v1:0"
  modelProvider: string;          // e.g., "bedrock"
  promptVersion: string;          // e.g., "interpret-v2"
  reasoning: string;              // The model's explanation
  rawResponse?: string;           // Full model response (for debugging)
}

interface InterpretationContext {
  currentDate: string;            // YYYY-MM-DD - runtime date for relative date inference
  // Future: location context, user preferences, etc.
}

interface SourceCost {
  modelCost: number;              // USD
  tokensIn: number;
  tokensOut: number;
  runtimeMs: number;
}

type InterpretationStatus =
  | 'draft'                       // Just created
  | 'pending_review'              // Awaiting human review
  | 'accepted'                    // Human accepted
  | 'challenged'                  // Human challenged
  | 'superseded'                  // Replaced by newer interpretation
  | 'parse_failed';               // LLM output could not be parsed

interface ExtractedDate {
  raw: string;                    // "Thursday 15th May"
  parsed: string;                 // "2026-05-15"
  confidence: 'certain' | 'inferred';
  inferenceReason?: string;       // "Year inferred as 2026"
}

interface Table {
  headers?: string[];
  rows: string[][];
  source: 'html' | 'spreadsheet' | 'ocr';
}
```

## ID Format

```
intp_[8-char-nanoid]

Examples:
intp_a1b2c3d4
intp_x9y8z7w6
```

## Runtime Context

The LLM receives **interpretation context** to ground its reasoning:

```
<context>
Current date: 2026-05-02
When inferring dates, assume events are in the future relative to the current date.
</context>
```

This prevents the model from defaulting to its training data's assumptions about dates. Without runtime context, "Thursday 15th May" might be interpreted as 2024 instead of 2026.

Future context may include:
- Geographic location hints
- Known venue/artist databases for matching
- User preferences

## Interpretation Lifecycle

```
Signal arrives
↓
Deterministic extraction runs
(dates, tables, text, OCR - cheap)
↓
LLM interpretation runs
(reasoning, claims - costs tracked)
↓
Interpretation v1 created
Status: PENDING_REVIEW
↓
Human reviews
├── Accept → Status: ACCEPTED
└── Challenge → Status: CHALLENGED
    └── May trigger re-interpretation (v2)
↓
Later: new evidence or model improvement
↓
Re-interpretation (v2)
Previous: Status: SUPERSEDED
```

## Cost Tracking

Every interpretation MUST record costs:

```typescript
sourceCost: {
  modelCost: 0.0023,      // $0.0023 USD
  tokensIn: 1547,
  tokensOut: 312,
  runtimeMs: 2340
}
```

This enables:
- Budget monitoring
- Cost per signal type analysis
- Model efficiency comparison
- Spend alerts

## Multiple Interpretations Example

```
Signal: Poster image (sgnl_abc123)

Interpretation v1 (intp_001):
  deterministicExtraction:
    ocrText: "STINGRAY LIVE AT THE RIGGER THURSDAY 15TH MAY"
  llmInterpretation:
    modelUsed: "eu.anthropic.claude-haiku-4-5-20251001-v1:0"
    reasoning: "Poster shows event announcement..."
  claims:
    - artist_performs: "Stingray"
    - venue_hosts: "The Rigger" (uncertain - no location)
    - event_date: "2026-05-15" (inferred from runtime date context)
  sourceCost:
    modelCost: 0.0036
    tokensIn: 1032
    tokensOut: 689
    runtimeMs: 3700
  status: ACCEPTED

--- Later: venue database updated ---

Interpretation v2 (intp_002):
  llmInterpretation:
    reasoning: "With venue match context, The Rigger is vnue_123"
  claims:
    - venue_hosts: vnue_123 (high confidence)
  status: ACCEPTED

Interpretation v1 → status: SUPERSEDED, supersededBy: intp_002
```

## DynamoDB Access Patterns

| Pattern | PK | SK |
|---------|----|----|
| Get interpretation | `INTP#intp_xxx` | `#METADATA` |
| List by signal | `SIGNAL#sgnl_xxx` | `INTP#intp_xxx` |
| List by status | GSI: `STATUS#pending_review` | `INTP#intp_xxx` |
| Get claims | `INTP#intp_xxx` | `CLAIM#claim_xxx` |

## Relationship to Other Entities

```
Signal (raw evidence)
└── Interpretation v1
    └── Claims
└── Interpretation v2
    └── Claims (refined)

EvidencePack
├── Signal 1 → Interpretation → Claims
├── Signal 2 → Interpretation → Claims
└── Corroboration across signals
```

## Error Handling

### Extraction Failures

If deterministic extraction fails (e.g., OCR error, empty image):
- Signal status set to `failed` with `failedStep: 'extraction'`
- Error recorded in DLQ for investigation
- No interpretation is attempted

### Parse Failures

If the LLM response cannot be parsed as structured JSON:
- Interpretation created with `status: 'parse_failed'`
- `rawResponse` and `parseError` preserved for debugging
- Signal status set to `failed` with `failedStep: 'interpretation'`
- Error thrown to trigger Step Functions failure handling

### Invalid Claims

If the LLM generates claims missing required fields (no object and no value):
- Claims are filtered but counted in `invalidClaimCount`
- Warning logged for debugging
- Noted in `uncertainties` array
- Valid claims proceed normally

This ensures failures are explicit, not silent.

## Re-interpretation Triggers

An interpretation may be superseded when:

| Trigger | Example |
|---------|---------|
| Human challenge | "That's not The Rigger" |
| New venue evidence | Venue database updated |
| New artist evidence | Artist confirmed identity |
| Model improvement | Better prompt or model version |
| Context change | Date passed, event confirmed |

## Related

- [[signal-model]] - Raw evidence input
- [[claim-model]] - What interpretations produce
- [[evidence-pack-model]] - Corroboration across signals
- [[../11-runtime/cognitive-runtime|Cognitive Runtime]] - How interpretations evolve

