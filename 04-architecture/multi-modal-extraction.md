# Multi-Modal Extraction (Bedrock/Claude Implementation)

Implementation details for using AWS Bedrock with Claude as the claim generator.

**This is one implementation option for the [[signal-reader|Signal Reader]] capability.**

The Signal Reader architecture is model-agnostic. This document covers the Claude/Bedrock implementation specifically.

## What Claude Vision Can Interpret

- Gig posters (photographed or designed)
- Screenshots of venue websites
- Facebook event images
- Flyers and handbills
- Calendar screenshots
- WhatsApp forwarded images

Same model that reads text also "sees" images.

## Real Example

**Input:** Poster image

**Claude sees:**
```
I can read this poster:

Artist: Kev Wright as Elvis Presley
Event type: Elvis Tribute
Venue: The Golden Lion, Pub Bedhampton Rd, Havant
Date: Friday 1st May @ 8:30pm

Claims I can make:
- Event exists on 1 May 2026
- Artist "Kev Wright" performs as Elvis tribute
- Venue is "The Golden Lion" in Havant
- Doors/start time is 8:30pm

Uncertainties:
- Year not explicit (inferred as 2026)
- Ticket price not visible
- Support acts not mentioned
```

## AWS Bedrock Integration

### Model Selection

| Model | Use Case | Cost | Accuracy |
|-------|----------|------|----------|
| **Claude Sonnet 4** | Complex posters, multiple events | $3/$15 per M tokens | High |
| **Claude Haiku 4** | Simple posters, high volume | $0.25/$1.25 per M tokens | Good |

**Default strategy:** Haiku first, escalate to Sonnet if confidence low.

### API Call Structure

```typescript
import { BedrockRuntimeClient, InvokeModelCommand } from "@aws-sdk/client-bedrock-runtime";

interface ExtractPosterRequest {
  signalId: string;
  imageBase64: string;
  mimeType: 'image/jpeg' | 'image/png' | 'image/webp' | 'image/gif';
}

async function extractFromPoster(request: ExtractPosterRequest): Promise<ClaimSet> {
  const client = new BedrockRuntimeClient({ region: 'eu-west-2' });

  const payload = {
    anthropic_version: "bedrock-2023-05-31",
    max_tokens: 1024,
    messages: [
      {
        role: "user",
        content: [
          {
            type: "image",
            source: {
              type: "base64",
              media_type: request.mimeType,
              data: request.imageBase64
            }
          },
          {
            type: "text",
            text: POSTER_EXTRACTION_PROMPT
          }
        ]
      }
    ]
  };

  const command = new InvokeModelCommand({
    modelId: "anthropic.claude-3-5-sonnet-20241022-v2:0",
    contentType: "application/json",
    body: JSON.stringify(payload)
  });

  const response = await client.send(command);
  return parseClaimsFromResponse(response);
}
```

### Prompt Template

```typescript
const POSTER_EXTRACTION_PROMPT = `
You are analyzing an image that appears to be promoting a live music event.

Your task is to identify what this image tells us about the grassroots music world.

Generate CLAIMS, not extracted fields. A claim is an interpretation of what the evidence suggests.

For each claim, indicate:
- What you're claiming (event exists, artist performs, venue hosts, etc.)
- The specific value (name, date, time, etc.)
- Your confidence: CLEAR (unambiguous), INFERRED (reasonable interpretation), UNCERTAIN (multiple possibilities)
- Any uncertainties or ambiguities

Format your response as JSON:
{
  "claims": [
    {
      "type": "event_exists",
      "value": "Elvis Tribute Night",
      "confidence": "CLEAR",
      "reasoning": "Title prominently displayed"
    },
    {
      "type": "artist_performs",
      "value": "Kev Wright",
      "confidence": "CLEAR",
      "reasoning": "Name at top of poster"
    },
    {
      "type": "venue_hosts",
      "value": "The Golden Lion",
      "location": "Bedhampton Rd, Havant",
      "confidence": "CLEAR",
      "reasoning": "Venue name and address visible"
    },
    {
      "type": "event_date",
      "value": "2026-05-01",
      "day": "Friday",
      "confidence": "INFERRED",
      "reasoning": "Shows 'Friday 1st May', year inferred as upcoming"
    },
    {
      "type": "event_time",
      "value": "20:30",
      "confidence": "CLEAR",
      "reasoning": "Shows '8:30pm'"
    }
  ],
  "uncertainties": [
    "Year not explicitly stated",
    "Ticket price not visible"
  ],
  "image_quality": "GOOD",
  "extraction_confidence": 0.85
}

If the image is not a music event poster, respond with:
{
  "claims": [],
  "not_music_event": true,
  "description": "What the image appears to be"
}
`;
```

## Image Preprocessing

### Before Sending to Bedrock

```typescript
interface ImagePreprocessing {
  maxDimension: 1568;          // Bedrock limit
  maxFileSizeKB: 5120;         // 5MB limit
  supportedFormats: ['jpeg', 'png', 'webp', 'gif'];
  convertTo: 'jpeg';           // Normalize format
  quality: 85;                 // Balance size/quality
}

async function preprocessImage(buffer: Buffer): Promise<ProcessedImage> {
  const sharp = require('sharp');

  const processed = await sharp(buffer)
    .resize(1568, 1568, {
      fit: 'inside',           // Maintain aspect ratio
      withoutEnlargement: true // Don't upscale
    })
    .jpeg({ quality: 85 })
    .toBuffer();

  return {
    base64: processed.toString('base64'),
    mimeType: 'image/jpeg',
    originalSize: buffer.length,
    processedSize: processed.length
  };
}
```

### Image Size vs Token Cost

| Image Size | Approximate Tokens | Sonnet Cost | Haiku Cost |
|------------|-------------------|-------------|------------|
| 400x400 | ~170 | $0.0005 | $0.00004 |
| 800x800 | ~680 | $0.002 | $0.0002 |
| 1200x1200 | ~1500 | $0.0045 | $0.0004 |
| 1568x1568 | ~2500 | $0.0075 | $0.0006 |

**Recommendation:** Resize to max 1200px for typical posters. Quality is sufficient, cost is lower.

## Cost Controls

### Per-Signal Budget

```typescript
interface CostControl {
  maxTokensPerSignal: 4000;    // Input + output combined
  maxCostPerSignal: 0.02;      // $0.02 USD
  monthlyBudget: 50.00;        // $50 USD
  alertThreshold: 0.80;        // Alert at 80% of budget
}
```

### Model Routing Strategy

```typescript
async function routeToModel(signal: Signal): Promise<ModelId> {
  // High-value signals get Sonnet
  if (signal.source === 'venue_owner' || signal.priority === 'high') {
    return 'anthropic.claude-3-5-sonnet-20241022-v2:0';
  }

  // Batch/scrape signals get Haiku
  if (signal.source === 'scraper' || signal.isAutomated) {
    return 'anthropic.claude-3-5-haiku-20241022-v1:0';
  }

  // Default to Haiku, escalate if needed
  return 'anthropic.claude-3-5-haiku-20241022-v1:0';
}

async function escalateIfNeeded(
  claims: ClaimSet,
  signal: Signal
): Promise<ClaimSet> {
  // If Haiku confidence is low, retry with Sonnet
  if (claims.extraction_confidence < 0.6 && !signal.escalated) {
    signal.escalated = true;
    return extractWithModel(signal, 'anthropic.claude-3-5-sonnet-20241022-v2:0');
  }
  return claims;
}
```

### Monthly Cost Projection

| Signal Type | Volume/Month | Model | Cost/Signal | Monthly Cost |
|-------------|--------------|-------|-------------|--------------|
| Poster uploads | 200 | Sonnet | $0.01 | $2.00 |
| Screenshots | 300 | Haiku | $0.001 | $0.30 |
| URL fetches | 1000 | Haiku | $0.0005 | $0.50 |
| Spreadsheets | 50 | Sonnet | $0.05 | $2.50 |
| Escalations | 100 | Sonnet | $0.01 | $1.00 |
| **Total** | **1650** | — | — | **~$6.30** |

Even at 10x volume: **~$63/month**

## Error Handling

### Failure Modes

| Error | Cause | Response |
|-------|-------|----------|
| `image_too_large` | Exceeds 5MB | Compress and retry |
| `invalid_format` | Unsupported type | Convert to JPEG |
| `content_policy` | Flagged content | Queue for human review |
| `rate_limited` | Too many requests | Exponential backoff |
| `model_error` | Bedrock issue | Retry with delay |
| `low_confidence` | Unclear image | Queue for human review |

### Retry Strategy

```typescript
const retryConfig = {
  maxRetries: 3,
  baseDelay: 1000,        // 1 second
  maxDelay: 30000,        // 30 seconds
  exponentialBase: 2,
  retryableErrors: [
    'ThrottlingException',
    'ServiceUnavailableException',
    'InternalServerException'
  ]
};
```

### Fallback to Human

```typescript
async function handleExtractionFailure(
  signal: Signal,
  error: ExtractionError
): Promise<void> {
  // Log the failure
  await logExtractionError(signal.signalId, error);

  // Update signal status
  await updateSignalStatus(signal.signalId, 'needs_human_review');

  // Queue for human review with context
  await queueForReview({
    signalId: signal.signalId,
    reason: 'extraction_failed',
    errorType: error.type,
    errorMessage: error.message,
    attemptCount: signal.extractionAttempts
  });
}
```

## Lambda Architecture

### Signal Processing Flow

```
S3 Event (new image uploaded)
↓
signal-processor Lambda
├── Fetch image from S3
├── Preprocess (resize, convert)
├── Route to model (Haiku/Sonnet)
├── Call Bedrock
├── Parse claims
├── Store claims in DynamoDB
├── Update signal status
└── Emit EventBridge event
```

### Lambda Configuration

```typescript
// signal-processor Lambda
const lambdaConfig = {
  runtime: 'nodejs20.x',
  memorySize: 1024,           // For image processing
  timeout: 60,                // Bedrock can be slow
  reservedConcurrency: 10,    // Limit parallel calls
  environment: {
    BEDROCK_REGION: 'eu-west-2',
    DEFAULT_MODEL: 'anthropic.claude-3-5-haiku-20241022-v1:0',
    ESCALATION_MODEL: 'anthropic.claude-3-5-sonnet-20241022-v2:0',
    MAX_IMAGE_DIMENSION: '1200',
    COST_LIMIT_PER_SIGNAL: '0.02'
  }
};
```

## Supported Image Types

| Type | Example | Extraction Quality |
|------|---------|-------------------|
| Digital poster | Designed flyer | Excellent |
| Photographed poster | Phone photo of poster | Good |
| Screenshot | Venue website capture | Excellent |
| Social media image | Facebook event graphic | Good |
| Handwritten | Chalk board menu | Fair |
| Low resolution | Thumbnail image | Poor |
| Multiple events | Calendar grid | Fair (per-cell) |

### Quality Assessment

```typescript
interface ImageQualityAssessment {
  resolution: 'high' | 'medium' | 'low';
  textClarity: 'clear' | 'readable' | 'blurry' | 'illegible';
  eventCount: number;         // How many events in image
  confidence: number;         // 0-1 overall extraction confidence
}
```

## Batch Processing

For spreadsheets or multi-event images:

```typescript
async function processBatchImage(signal: Signal): Promise<ClaimSet[]> {
  // First pass: identify how many events
  const overview = await extractOverview(signal);

  if (overview.eventCount === 1) {
    return [await extractSingleEvent(signal)];
  }

  // Multiple events: extract each
  const claims: ClaimSet[] = [];
  for (let i = 0; i < overview.eventCount; i++) {
    const eventClaims = await extractEventAtIndex(signal, i, overview);
    claims.push(eventClaims);
  }

  return claims;
}
```

## Monitoring

### CloudWatch Metrics

| Metric | Description | Alert Threshold |
|--------|-------------|-----------------|
| `ExtractionSuccessRate` | % successful extractions | < 85% |
| `AverageConfidence` | Mean extraction confidence | < 0.7 |
| `ModelLatency` | Bedrock response time | > 10s |
| `CostPerSignal` | Average USD per signal | > $0.05 |
| `EscalationRate` | % escalated to Sonnet | > 30% |
| `HumanReviewRate` | % sent to human queue | > 20% |

### Cost Dashboard

```
Daily Bedrock Costs
├── Haiku: $X.XX (N signals)
├── Sonnet: $X.XX (N signals)
├── Escalations: $X.XX (N signals)
└── Total: $X.XX

Monthly Projection: $XX.XX
Budget Remaining: $XX.XX
```

## Related

- [[ingestion-pipeline]] - Overall pipeline architecture
- [[../05-entities/signal-model|Signal Model]] - Signal entity definition
- [[../10-brain/signal-to-claim-model|Signal to Claim]] - How claims are generated
- [[../03-backlog/build-plan|Build Plan]] - Implementation phases

