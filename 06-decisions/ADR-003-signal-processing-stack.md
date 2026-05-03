# ADR-003: Signal Processing Stack

---

## Status

**Status:** Accepted
**Date:** 03/05/2026
**Deciders:** Jason, Claude
**Supersedes:** None

## Context

bndy needs to process "signals" (user-submitted evidence about live music events) and extract structured claims. The system must:

- Accept multiple input types (text, images, URLs, spreadsheets)
- Extract text from images (OCR)
- Use AI to interpret content and generate claims
- Track costs per interpretation
- Support human review before publishing

Forces at play:
- **Cost control**: AI interpretation is expensive; need per-request tracking
- **EU data residency**: Preference for EU-based services
- **Speed**: Users expect fast feedback after submitting
- **Accuracy**: Need to handle messy real-world data (posters, Facebook posts)
- **Maintainability**: Small team, prefer managed services

## Decision

We decided to use **AWS Step Functions + Textract + Bedrock (Claude Haiku 4.5)** because:

> In the context of signal processing for live music events,
> facing the need for multi-modal extraction with cost control,
> we decided to use AWS Step Functions orchestrating Textract OCR and Bedrock LLM,
> to achieve reliable, cost-tracked interpretation with EU inference,
> accepting AWS vendor lock-in and Textract's limitations on stylized text.

### Specific choices:

| Component | Choice | Reason |
|-----------|--------|--------|
| Orchestration | Step Functions | Visual workflow, built-in retry/error handling |
| OCR | Textract | Managed, no infrastructure, fast |
| LLM | Bedrock Claude Haiku 4.5 | EU inference profile, cost-effective, good reasoning |
| Model ID | `eu.anthropic.claude-haiku-4-5-20251001-v1:0` | EU data residency |
| Storage | S3 + DynamoDB | Already using for v1, familiar |

## Consequences

### Positive

- **Cost visibility**: Every interpretation records tokensIn, tokensOut, modelCost, runtimeMs
- **EU compliance**: Bedrock EU inference profile keeps data in EU
- **Reliability**: Step Functions handles retries, DLQ for failures
- **Fast iteration**: Change prompt without redeploying infrastructure
- **Extensible**: Easy to add URL rendering, spreadsheet parsing later

### Negative

- **Textract limitations**: Struggles with artistic fonts, low-contrast posters
- **Cold starts**: Lambda cold starts add latency
- **Bedrock pricing**: ~$0.004 per interpretation adds up at scale
- **Vendor lock-in**: Deep integration with AWS services

### Neutral

- Model can be switched via environment variable
- Pricing rates configurable via env vars

## Alternatives Considered

### Option 1: OpenAI GPT-4o Vision

Use GPT-4o with vision directly on images.

**Pros:**
- Single API call for OCR + interpretation
- Better at stylized text

**Cons:**
- No EU inference option
- Higher cost per call
- Harder to separate extraction costs from interpretation costs

**Why rejected:** EU data residency requirement, cost control needs separate OCR step.

### Option 2: Self-hosted Tesseract + local LLM

Run OCR locally, use open-source LLM.

**Pros:**
- No per-request costs
- Full control

**Cons:**
- Infrastructure overhead
- Quality issues with both OCR and LLM
- Team doesn't have ML ops expertise

**Why rejected:** Operational complexity outweighs cost savings at current scale.

### Option 3: Google Cloud Vision + Gemini

Use Google's stack instead of AWS.

**Pros:**
- Good OCR quality
- Gemini is competitive with Claude

**Cons:**
- Would need to set up new cloud account
- Team more familiar with AWS
- Already invested in AWS infrastructure

**Why rejected:** Switching costs not justified, AWS stack is working.

## Related

- [[../03-backlog/now|Now]] - Current build status
- [[../05-entities/interpretation-model|Interpretation Model]] - How interpretations work
- [[../11-runtime/cognitive-runtime|Cognitive Runtime]] - The reasoning layer
