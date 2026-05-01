# Signal Reader

The system capability that turns raw signals into claims.

## What Signal Reader Is

A composable pipeline that takes any signal type and outputs claims.

```
Signal (URL / image / text / XLS)
↓
Renderer (optional)
↓
Extractor
↓
Claim Generator
↓
Claims
```

**Signal Reader is a capability, not a product.**

The underlying models, renderers, and extractors are implementation choices.

## Pipeline Stages

### 1. Renderer

For signals that need rendering before extraction.

| Signal Type | Renderer | Output |
|-------------|----------|--------|
| URL | Playwright | Visible text + DOM + screenshot + asset URLs |
| Dynamic page | Playwright | Rendered HTML after JS execution |
| PDF | PDF parser | Text + images per page |
| XLS/CSV | Table parser | Structured rows |
| Image | None | Pass through |
| Text | None | Pass through |

**Why Playwright for URLs:**

Many venue sites (like ScenicEye) render content via JavaScript. Raw HTML fetch misses:
- Dynamically loaded events
- Calendar widgets
- Embedded images
- Interactive elements

Playwright renders the page as a browser would, then extracts:
- Visible text
- DOM structure
- Screenshot
- Image URLs

### 2. Extractor

Transforms rendered content into extractable assets.

| Content Type | Extractor | Output |
|--------------|-----------|--------|
| Rendered HTML | DOM parser | Text blocks, tables, lists |
| Screenshot | OCR | Text regions with coordinates |
| Image | Vision preprocessor | Normalized image for model |
| Table | Structure parser | Row/column data |
| Text | Tokenizer | Clean text |

**Extractor outputs:**
```typescript
interface ExtractedAssets {
  textBlocks: TextBlock[];
  tables: Table[];
  images: ImageAsset[];
  metadata: {
    title?: string;
    url?: string;
    capturedAt: string;
  };
}
```

### 3. Claim Generator

Interprets extracted assets and generates claims.

**Input:** Extracted assets (text, images, tables)
**Output:** Claims with confidence and uncertainties

The claim generator is model-agnostic. It can use:
- Claude (via Bedrock)
- GPT-4 (via OpenAI)
- Gemini (via Google)
- Local models (Llama, etc.)
- Hybrid approaches

**The architecture does not prescribe which model.**

## Signal Type Flows

### URL Signal

```
URL: https://scenicmind.co.uk/sceniceye
↓
Playwright render (headless browser)
↓
Extract: visible text + DOM + screenshot
↓
OCR screenshot (if needed for images)
↓
Claim generator interprets content
↓
Claims: events, venues, artists, dates
```

### Image Signal

```
Image: poster.jpg
↓
Preprocess: resize, normalize
↓
OCR: extract text regions
↓
Vision model: interpret layout + meaning
↓
Claim generator
↓
Claims
```

### Text Signal

```
Text: Facebook event paste
↓
Clean/normalize text
↓
Claim generator interprets
↓
Claims
```

### Spreadsheet Signal

```
XLS: venue_gig_list.xlsx
↓
Table parser: extract rows/columns
↓
Identify structure: date column, venue column, artist column
↓
Claim generator per row
↓
Claims (batch)
```

## Component Interfaces

### Renderer Interface

```typescript
interface Renderer {
  canRender(signal: Signal): boolean;
  render(signal: Signal): Promise<RenderedContent>;
}

interface RenderedContent {
  html?: string;
  text: string;
  screenshot?: Buffer;
  assets: AssetReference[];
  metadata: RenderMetadata;
}
```

### Extractor Interface

```typescript
interface Extractor {
  canExtract(content: RenderedContent): boolean;
  extract(content: RenderedContent): Promise<ExtractedAssets>;
}

interface ExtractedAssets {
  textBlocks: TextBlock[];
  tables: Table[];
  images: ImageAsset[];
  metadata: ExtractionMetadata;
}
```

### Claim Generator Interface

```typescript
interface ClaimGenerator {
  generate(assets: ExtractedAssets): Promise<ClaimSet>;
}

interface ClaimSet {
  claims: Claim[];
  uncertainties: string[];
  confidence: number;
  modelUsed?: string;  // Implementation detail
}
```

## Implementation Options

### Renderer Implementations

| Implementation | Use Case | Notes |
|----------------|----------|-------|
| Playwright | JavaScript-heavy sites | Full browser rendering |
| Puppeteer | Simpler sites | Chromium only |
| Simple fetch | Static HTML | Fastest, limited |
| PDF.js | PDF documents | Text + image extraction |
| SheetJS | Excel/CSV | Table parsing |

### OCR Implementations

| Implementation | Use Case | Notes |
|----------------|----------|-------|
| Tesseract | Open source, self-hosted | Good accuracy |
| AWS Textract | AWS-native | Better accuracy, cost |
| Google Vision | GCP | High accuracy |
| Model-native OCR | Claude/GPT vision | Built-in, convenient |

### Claim Generator Implementations

| Implementation | Use Case | Notes |
|----------------|----------|-------|
| Claude (Bedrock) | AWS-native, multi-modal | See [[multi-modal-extraction]] |
| GPT-4 (OpenAI) | Alternative provider | Similar capability |
| Gemini (Google) | GCP-native | Multi-modal |
| Local model | Cost control, privacy | Lower accuracy |

**The Signal Reader architecture supports any combination.**

## Rendering Strategy for URLs

### ScenicEye Example

```
URL: https://scenicmind.co.uk/sceniceye
↓
Playwright launches headless Chromium
↓
Navigate to URL, wait for content
↓
Extract:
├── Visible text (event listings)
├── DOM structure (dates, venues, artists)
├── Screenshot (full page)
└── Image URLs (poster thumbnails)
↓
For each poster image URL:
├── Fetch image
├── OCR + vision
└── Extract additional claims
↓
Merge all claims
↓
Output: 40+ events with venues, artists, dates
```

### Playwright Configuration

```typescript
interface PlaywrightConfig {
  headless: true;
  timeout: 30000;              // 30 second page load
  waitUntil: 'networkidle';    // Wait for JS to settle
  viewport: { width: 1280, height: 720 };
  userAgent: 'bndy-signal-reader/1.0';

  // Extraction options
  extractText: true;
  extractScreenshot: true;
  extractImages: true;
  maxImages: 20;               // Limit image fetches
}
```

## Cost Model

Separate costs for each stage:

| Stage | Cost Driver | Typical Cost |
|-------|-------------|--------------|
| Renderer | Compute time | $0.001/page |
| OCR | API calls or compute | $0.001-0.01/image |
| Claim Generator | Model tokens | $0.001-0.01/signal |

**Total per URL signal:** ~$0.005-0.02
**Total per image signal:** ~$0.002-0.01

## Error Handling

| Stage | Error | Response |
|-------|-------|----------|
| Renderer | Timeout | Retry with longer timeout, then fail |
| Renderer | Blocked | Try different user agent, then fail |
| Extractor | Parse error | Fall back to raw text |
| OCR | Low confidence | Flag for human review |
| Claim Generator | Model error | Retry, then queue for review |

## Monitoring

| Metric | Description |
|--------|-------------|
| `render_success_rate` | % of URLs rendered successfully |
| `extraction_quality` | Mean text extraction confidence |
| `claim_generation_rate` | Claims per signal |
| `pipeline_latency` | End-to-end processing time |
| `cost_per_signal` | Total cost across all stages |

## Lambda Architecture

```
S3 Event (new signal)
↓
signal-reader Lambda
├── Route to appropriate renderer
├── Render signal
├── Extract assets
├── Route to claim generator
├── Store claims
└── Update signal status
↓
EventBridge (claims generated)
```

**Memory/timeout considerations:**
- Playwright needs ~512MB+ memory
- Page renders can take 10-30 seconds
- Consider Step Functions for complex pipelines

## Related

- [[ingestion-pipeline]] - Overall pipeline architecture
- [[multi-modal-extraction]] - Claude/Bedrock implementation details
- [[../05-entities/signal-model|Signal Model]] - Signal entity definition
- [[../10-brain/signal-to-claim-model|Signal to Claim]] - Claim generation model
- [[../03-backlog/build-plan|Build Plan]] - Implementation phases

