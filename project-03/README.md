# Project 03: Competitor Monitoring AI Agent

**Externship:** Wayfair AI Automation via Extern  
**Category Focus:** Area Rugs  
**Workflow:** Competitor Monitoring Agent  
**Status:** Complete — All Stages Finalized · Report Delivered

---

## Overview

A multi-stage n8n automation agent that benchmarks Wayfair's Area Rug catalog against key competitors — Amazon, Walmart, and Target — by scraping live product data, normalizing attributes, and running a sequential AI analysis pipeline. The final output is a styled, decision-ready HTML competitive intelligence report uploaded directly to Google Drive.

**The problem it solves:** Writing a competitive analysis manually requires hours of browsing competitor pages, tracking pricing and feature shifts, and organizing findings into a coherent report. This agent automates the entire workflow — input a rug category and competitor URLs, and receive a structured report covering pricing gaps, feature gaps, assortment opportunities, and strategic recommendations in 15–20 minutes.

---

## Build Progress

| Stage | Description | Status |
|---|---|---|
| **Stage 1** | Input & Routing — Chat Trigger + Input Parser | ✅ Complete |
| **Stage 2** | Wayfair Baseline Scraper | ✅ Complete |
| **Stage 3** | Amazon Scraper (dual-path: collection + direct) | ✅ Complete |
| **Stage 4** | Walmart Scraper (dual-path: collection + direct) | ✅ Complete |
| **Stage 5** | AI Analysis Pipeline — 7-section sequential report | ✅ Complete |
| **Stage 6** | HTML Report Assembly & Download | ✅ Complete |
| **Stage 7** | Google Drive Upload | ✅ Complete |
| **Target Scraper** | Target competitor scraper | ⏳ Deferred to v2 |

**Workflow stats:** 65 nodes · 56 connections · 2 retailers live · 6 AI agents · last updated May 2026

---

## Why Competitor Monitoring?

Project 02 answered: *"What are consumers trending toward?"*  
Project 03 answers: *"What are competitors already doing about it?"*

At Wayfair, competitive intelligence directly informs:

| Business Function | What the agent surfaces |
|---|---|
| Assortment planning | Features competitors carry that Wayfair lacks |
| Pricing strategy | Price bands where competitors undercut or Wayfair is exposed |
| Visual merchandising | How competitors position and merchandise the same products |
| Supplier sourcing | Brands appearing on competitors not yet on Wayfair |
| Risk management | Competitor moves that threaten Wayfair's positioning |

A single insight can drive real decisions — if Amazon and Walmart are aggressively pushing washable rugs at $25–$65 with OEKO-TEX credentialing, Wayfair's category team needs to know whether to match on features, differentiate on design, or reprice. This agent surfaces that signal automatically.

---

## Competitor Selection — Strategic Rationale

### Included in v1

| Retailer | Why included |
|---|---|
| **Amazon** | Wayfair's most direct e-commerce competitor; strongest proxy for real-time demand signals; high SKU volume with scraped price, material, and feature data |
| **Walmart** | Growing design-conscious assortment (SAFAVIEH, Mainstays); aggressive discounting in the $27–$70 budget tier; most direct threat on washable/functional rugs |
| **Target** | Threshold and Studio McGee brands compete on Wayfair's exact design-positioning turf in the $40–$150 tier; most strategically important gap in the original Amazon + Walmart analysis |

### Deferred to v2

| Retailer | Why deferred |
|---|---|
| **IKEA** | Online catalog is shallow relative to in-store depth; primary competitive threat is the physical discovery experience which no web scraper can capture; site structure requires custom parsing logic for comparatively low signal return |
| **Ruggable** | D2C brand, no traditional PDP scraping path; best monitored via social signals and press rather than product listing scraping |

---

## Full Workflow Architecture

```
[STAGE 1] User Input
  Chat Trigger
    └── Category name + Amazon URLs + Walmart URLs + Target URLs

[STAGE 2] Input Parsing & Validation
  Input Parser (Code)
    └── Category Validator (IF)
          ├── ✅ Valid → fan out to scraping stages
          └── ❌ Invalid → Error Response (stop workflow)

[STAGE 3] Wayfair Scraping — Baseline
  Prepare Wayfair URLs → Is List Empty? (IF)
    └── Visit Product Page (HTTP) → Parse Wayfair Details (Cheerio)
          └── Deduplicate + Build Wayfair Product List

[STAGE 4] Amazon Scraping — Competitor
  Path 1 (Collection URLs):
    Prepare Collection URLs → Is List Empty? (IF)
      └── Visit Aisle Page (HTTP) → Extract Product Links (HTML)
            → Remove Duplicates → Pause (1.5s Wait)
              → Visit Product Page (HTTP) → Read Item Details (Cheerio)

  Path 2 (Direct URLs):
    Prepare Direct Links → Do We Have Direct Links? (IF)
      └── Visit Direct Product Page (HTTP) → Read Direct Item Details (Cheerio)

  Both paths → Merge Amazon Products → Deduplicate + Combine Final List

[STAGE 5] Walmart Scraping — Competitor
  (Same dual-path pattern as Stage 4, parameterized for Walmart)

[STAGE 5B] Target Scraping — Competitor (v1 addition)
  (Same dual-path pattern as Stage 4, parameterized for Target)

[STAGE 6] AI Analysis — Sequential Pipeline
  Merge All Retailer Products
    └── Wait
          └── Scope Generator (Code)
                └── Wait → Executive Summary (Gemini Agent)
                      └── Wait → Competitor Analysis (Gemini Agent)
                            └── Wait → Comparison (Gemini Agent)
                                  └── Wait → Pricing & Whitespace (Gemini Agent)
                                        └── Wait → Recommendations (Gemini Agent)
                                              └── Wait → Supplier ID (Gemini Agent)

[STAGE 7] Report Assembly
  Collect All Sections (Code)
    └── Assemble HTML Report (Code)
          └── Section Validator (Code)
                └── Create Download File (Code)

[STAGE 8] Google Drive Upload
  Upload to Wayfair Reports folder (Google Drive MCP)
    └── Competitive Intelligence Report — styled HTML · downloadable
```

---

## Stage-by-Stage Breakdown

### Stage 1 — User Input

- **Trigger:** Chat message containing category name + retailer URLs
- **Accepted categories:** `area_rug`, `outdoor_rug`, `hallway_runner`, `shag_rug`
- **Test message format:**
```
Area Rug - 
https://www.wayfair.com/rugs/... 
https://www.amazon.com/s?k=area+rugs 
https://www.amazon.com/dp/[ASIN]
https://www.walmart.com/browse/area-rugs/...
https://www.target.com/c/rugs/-/N-5xtnr...
```

---

### Stage 2 — Input Parsing & Validation

- **Input Parser (Code):** Extracts `selectedCategory`, `categoryKey`, and URL arrays keyed by retailer — `wayfairUrls[]`, `amazonUrls[]`, `walmartUrls[]`, `targetUrls[]`
- **Category Validator (IF):** Checks `isValid === true` before any scraping begins — prevents unnecessary HTTP calls on malformed input
- **Error path:** Returns a descriptive message explaining what was missing or invalid

---

### Stage 5 — Walmart Scraping: ScraperAPI Integration

Walmart's anti-bot infrastructure returns a **412 Precondition Failed** response to standard HTTP requests regardless of User-Agent headers. Direct fetch is blocked at the TLS fingerprinting layer — automated clients cannot replicate the browser handshake signals Walmart checks.

**Fix applied:** All three Walmart HTTP nodes (`Fetch Walmart Product`, `Fetch Walmart Collection`, `Fetch Walmart Product2`) are routed through ScraperAPI, which handles rotating residential proxies, browser rendering, and bot-detection bypass automatically.

```
// URL field in each Walmart HTTP node
http://api.scraperapi.com?api_key={{ $credentials.scraperApiKey }}&url={{ encodeURIComponent($json.url) }}
```

| Consideration | Detail |
|---|---|
| Latency | +3–8s per request vs. <1s direct fetch |
| Rate limit | Free tier: 1,000 calls/month |
| Reliability | More reliable than direct fetch once bot detection triggers |
| Mitigation | Existing 1.5s Wait nodes kept alongside ScraperAPI routing |

---

### Stages 3–5 — Scraping Layer (Modular Pattern)

All retailer scrapers follow the same parameterized template:

1. Prepare URLs → split into individual items
2. Guard against empty input (IF node) — prevents null HTTP requests
3. Visit page with browser headers to mimic human traffic
4. Parse HTML with Cheerio — extract: `name`, `price`, `material`, `pile_height`, `size`, `features[]`, `rating`, `review_count`, `url`
5. Apply **1.5s Wait** between requests to avoid rate limiting
6. Deduplicate by URL → produce clean `retailerProducts[]` array

**Standardized feature terms** used across all scrapers:
`machine_washable` · `non_slip` · `oeko_tex` · `indoor_outdoor` · `stain_resistant` · `pet_friendly` · `washable`

---

### Stage 6 — AI Analysis Pipeline

Seven sequential Gemini agents, each with a focused single-job system message. **Wait nodes between every call** to respect Gemini rate limits.

| Node | Model | Output |
|---|---|---|
| **Scope Generator** | Code | Research scope metadata — date, category, retailers, product counts |
| **Executive Summary** | Gemini | Competitive position, top threat, top opportunity, key stats |
| **Competitor Analysis** | Gemini | Per-retailer: strategy label, feature advantages, Wayfair strengths, threat level |
| **Comparison** | Gemini | Head-to-head feature matrix across all retailers |
| **Pricing & Whitespace** | Gemini | Price band heatmap, up to 4 whitespace opportunities, risk matrix |
| **Recommendations** | Gemini | Quick wins (0–30d), near-term (30–90d), strategic (90–180d), watch items |
| **Supplier ID** | Gemini | Brands on competitors not on Wayfair; sourcing candidates |

**System message architecture — 5-layer framework applied to every node:**

| Layer | Purpose | Why it matters |
|---|---|---|
| **Role** | "Senior category analyst at Wayfair..." | Anchors reasoning style — forces domain-specific, VP-ready output |
| **Context** | Explicit field names and data shape | Prevents hallucination of missing fields the AI can't see |
| **Task** | One numbered task per node | Single-job prompts produce sharper output than multi-task prompts |
| **Output format** | Exact JSON skeleton with field names | Downstream Code nodes depend on consistent schema every run |
| **Guardrails** | "Return JSON only. No markdown. No preamble." | Prevents Gemini wrapping output in ```json``` fences — the #1 workflow stall |

**Key guardrail pattern applied to all nodes:**
```
GUARDRAILS:
- Return ONLY valid JSON. No preamble, no explanation, no markdown fences.
- Every claim must be traceable to a product in the input data.
- Do NOT invent products, prices, or features not present in the data.
- If a retailer has fewer than 3 products, flag low_sample: true.
```

**n8n expression used to inject live category context:**
```
{{ $('Input Parser').first().json.selectedCategory }}
```
This scopes every AI analysis to the user's actual input — no hardcoded category names, one workflow handles all rug types.

---

### Stage 7 — Report Assembly

- **Collect All Sections:** Gathers all Gemini outputs into a single object
- **Assemble HTML Report:** Applies global CSS, injects all sections in order, embeds any generated visuals
- **Section Validator:** Confirms all key sections are present; flags missing sections with warnings rather than failing silently
- **Create Download File:** Converts final HTML to binary with filename `area-rug-competitor-report-YYYY-MM-DD.html`

---

### Stage 8 — Google Drive Upload

- Uploads the compiled HTML report to the **Wayfair Reports** folder in Google Drive
- Uses Google Drive MCP credentials configured in n8n
- Returns a shareable link in the workflow output

> **Note:** This stage requires Google Drive MCP credentials to be active in n8n before the final workflow run.

---

## Models & Tools

| Purpose | Tool |
|---|---|
| Competitive analysis, report writing | Google Gemini API |
| Workflow automation | n8n |
| Product data scraping | HTTP Request + Cheerio (JavaScript) |
| Report delivery | Google Drive MCP |
| Version control | GitHub |

**API Credentials required:**
- Google Gemini API
- Google Drive MCP (OAuth)
- ScraperAPI key (for Walmart scraping — store as n8n credential, do not hardcode in URL field)

---

## Architectural Decisions & Rationale

### 1. Modular, parameterized scrapers
Each retailer scraper (Stages 3–5B) uses the same HTTP + Cheerio parsing template with retailer-specific selectors injected as parameters. Adding a new competitor is a configuration change, not a rebuild — new retailer slots into the merge stage without touching existing scraper logic.

### 2. One AI node, one job
Each Stage 6 sub-node owns exactly one report section. Asking a single prompt to produce multiple sections degrades output quality across all of them. Focused prompts produce consistent, parseable JSON.

### 3. Strict JSON-only output enforced at the prompt level
Gemini defaults to wrapping output in markdown code fences (` ```json ``` `) when not explicitly constrained. This silently breaks `JSON.parse()` in downstream Code nodes. The guardrail `"Return JSON only. No markdown. No preamble."` is non-negotiable on every AI node — a lesson carried forward from Project 02.

### 4. `low_sample` flag for data quality transparency
If a retailer returns fewer than 3 products, nodes flag `low_sample: true` in the JSON output rather than silently extrapolating from insufficient data. The report assembler surfaces this as a visible caveat in the final HTML.

### 5. Wait nodes are load-bearing
Gemini rate limits will fail chained AI calls without deliberate pauses. Every Stage 6 node is preceded by a Wait node — this is not optional. Runtime increases but reliability across the full pipeline is maintained.

### 6. ScraperAPI as the Walmart proxy layer
Direct HTTP requests to Walmart are blocked by TLS fingerprinting and bot detection — User-Agent spoofing alone is insufficient. Routing through ScraperAPI adds 3–8s per request but is the only reliable fetch path. This is treated as an infrastructure dependency, not a workaround.

### 7. Null-safe empty returns in data pipeline nodes
Code nodes that split URL arrays into individual items return `[]` (empty array) when the input array is empty — not a placeholder item with `url: null`. A null URL passing through an IF gate with strict `notEquals ""` type validation evaluates as truthy (`null !== ""`), leaks into the HTTP Request node, and is stringified to the literal string `"null"` by n8n's expression engine, causing a "URL must start with http" error. Empty array stops the chain cleanly with zero items.

---

## Pre-Build Competitive Intelligence Findings

Before building, a manual competitor analysis was run on two sample products to validate the agent's required output:

**Wayfair sample:** Bungalow Rose Ornate Lamotte Indoor/Outdoor Area Rug (~$89, 5'x8')  
**Amazon sample:** Yinhua Cowhide Faux Western Area Rug (~$29–$55, 4x6–8x10)

| Dimension | Wayfair | Amazon |
|---|---|---|
| Price range | ~$60–$150 | ~$25–$65 |
| Material | Polypropylene, OEKO-TEX available | Faux wool, OEKO-TEX certified |
| Non-slip | Requires separate rug pad | Built-in TPE backing |
| Machine washable | Available but not prominently surfaced | Front-of-listing headline feature |
| Merchandising | Editorial lifestyle photography | Feature keyword-led titles |
| Notable gap | No built-in non-slip on sampled SKU | Lower price, full washability baked in |

**Manager takeaway:** Amazon is commoditizing the washable, non-slip area rug segment at $25–$65 with OEKO-TEX credentialing. Wayfair has the inventory to compete — but washability and non-slip must be promoted as front-of-page assortment features, not buried on the PDP.

---

## Key Learnings (Pre-Build)

- **Modular scraper design pays off on day one:** Reusing the Cheerio parsing pattern from Project 02 for three new retailers took hours instead of days — the template transfers directly
- **Competitor selection is a strategic decision, not just a technical one:** Target was chosen because it competes on design positioning, not just price — Amazon and Walmart alone would miss Wayfair's most direct threat in the $50–$150 tier
- **System message quality determines report quality more than model choice:** A well-structured 5-layer system message outperforms a stronger model with a vague prompt — the framework matters more than the API
- **JSON schema drift is a silent killer:** If field names in the AI output differ between runs, the report assembler fails with no obvious error. Hardcoding the exact schema in the system message is the only reliable fix

## Key Learnings (Build — Stages 1–5)

- **n8n expression format is load-bearing:** `=={{$json.url}}` (no space) causes n8n to prepend a literal `=` to every resolved URL — the sentinel `=` prefix strips correctly only when the expression body starts with `{{ ` (space after the opening braces). Every HTTP node URL field must use `={{ $json.url }}`, not `=={{$json.url}}`. One character difference, workflow-breaking result.
- **Null URLs become the string "null" in n8n expressions:** When `$json.url` is JavaScript `null` and evaluated inside `={{ }}`, n8n stringifies it to the four-character string `"null"`. An IF node checking `url notEquals ""` with strict type validation passes null through because `null !== ""` is true. Fix: return `[]` from Code nodes when input arrays are empty — zero items, zero downstream execution.
- **Bot detection requires infrastructure-level bypass, not header tricks:** Walmart's 412 response is triggered at the TLS layer, not the application layer. Rotating User-Agent headers has no effect. ScraperAPI is the correct solution — it should be treated as a required credential alongside the API keys, not a workaround added after the fact.
- **Modular scraper patterns expose bugs faster:** Building Walmart scraping as a parameterized copy of the Amazon pattern meant the same expression bug appeared in both — and was caught and fixed in both simultaneously. Consistency in node patterns makes debugging O(1) instead of O(n retailers).

---

## Planned Improvements

- [ ] Add IKEA as a v2 competitor scraper — requires custom parsing logic for their non-standard PDP structure; document here when implemented
- [ ] Add `low_sample` warning banners to the HTML report output so end readers see data quality caveats inline
- [ ] Build a change-detection layer: compare this run's output to the previous report stored in Google Drive and flag what changed (new products, price shifts, feature additions)
- [ ] Expand scraping to capture review sentiment as a signal — product ratings alone miss the "why" behind customer satisfaction gaps

---

## Files

| File | Description |
|---|---|
| `README.md` | Project documentation (this file) |
| `Competitor_Analysis_Report.html` | Final deliverable — styled HTML competitive intelligence report (Wayfair vs Amazon vs Walmart, Area Rugs). Sections: executive summary, competitor analysis, comparison table, pricing insights, whitespace analysis, strategic recommendations, supplier identification |
| `Competitor_Analysis_Report.pdf` | PDF export of the final report — print-ready format for sharing with stakeholders |
| `Wayfair_Competitor_Monitoring_Agent_Workflow_SAFE.json` | n8n workflow export — all credentials removed, session cookies replaced with env variable references |
| `.env.example` | Environment variable template — copy to `.env` and fill in values; never committed to GitHub |
| `.gitignore` | Blocks `.env`, credential exports, and OS/editor artifacts from being committed |
| `Project03_Error_Log.docx` | Error log — all bugs encountered and resolved during build |

---

*Wayfair AI Automation Externship · Extern · May 2026*
