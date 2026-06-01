# Project 03: Competitor Monitoring AI Agent

**Externship:** Wayfair AI Automation via Extern  
**Category Focus:** Area Rugs  
**Workflow:** Competitor Monitoring Agent  
**Status:** Complete — All Stages Finalized · Report Delivered

---

## Overview

A multi-stage n8n automation agent that benchmarks Wayfair's Area Rug catalog against key competitors — Amazon and Walmart — by scraping live product data, normalizing attributes, and running a sequential AI analysis pipeline. The final output is a styled, decision-ready HTML competitive intelligence report uploaded directly to Google Drive.

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

**Workflow stats:** 88 nodes · 81 connections · 2 retailers live · 6 AI agents · last updated May 2026

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

### Deferred to v2

| Retailer | Why deferred |
|---|---|
| **Target** | Threshold and Studio McGee brands compete on Wayfair's exact design-positioning turf in the $40–$150 tier; most strategically important gap in the Amazon + Walmart analysis — added once dual-path scraper pattern is validated on live retailers |
| **IKEA** | Online catalog is shallow relative to in-store depth; primary competitive threat is the physical discovery experience which no web scraper can capture; site structure requires custom parsing logic for comparatively low signal return |
| **Ruggable** | D2C brand, no traditional PDP scraping path; best monitored via social signals and press rather than product listing scraping |

---

## Full Workflow Architecture

```
[STAGE 1] Input & Routing
  Chat Trigger (When chat message received)
    └── Input Parser (Code)
          └── If (Category Validator)
                ├── ✅ Valid → fan out to scraping stages
                └── ❌ Invalid → Error Response (stop workflow)

[STAGE 2] Wayfair Scraping — Baseline
  Fetch Wayfair Rug Page (HTTP)
    └── Extract Rug Products (Code)
          └── Filter Valid Products (Filter)
                └── Create Final JSON (Code)
                      └── Code (debug pass-through)

[STAGE 3] Amazon Scraping — Competitor
  Path 1 (Collection URLs):
    Prepare Collection List (Code) → Is List Empty? (IF)
      └── Visit Aisle Page (HTTP) → Extract Product Links (HTML/Cheerio)
            → Remove Duplicate Links (Code) → Pause for Safety (Wait 1.5s)
              → Visit Product Page (HTTP) → Read Item Details (Code)

  Path 2 (Direct URLs):
    Prepare Direct Links (Code) → Do We Have Direct Links? (IF)
      └── Visit Direct Product Page (HTTP) → Read Direct Item Details (Code)

  Both paths → Merge Amazon Products → Combine Amazon Products (Code)

[STAGE 4] Walmart Scraping — Competitor
  Product path:
    Process Walmart Products (Code) → Has Walmart Product URL? (IF)
      └── Wait Walmart Product → Fetch Walmart Product (HTTP)
            → Parse Walmart Product (Code)

  Collection path:
    Process Walmart Collections (Code) → Has Walmart Collection URL? (IF)
      └── Fetch Walmart Collection (HTTP) → Extract Walmart Links (Code)
            → Wait Walmart Collection → Fetch Walmart Product (HTTP)
              → Parse Walmart Product (Code)

  Both paths → Merge Walmart Results → Combine Walmart Products (Code)

  Merge (all retailers: Wayfair + Amazon + Walmart)

[STAGE 5] AI Analysis — Sequential Pipeline
  Scope Generator (Code — static HTML section)
    └── Wait After Scope
          └── Executive Summary Generator    (Gemini Agent)
                └── Wait After Executive
                      └── Amazon Analysis Generator      (Mistral Agent)
                            └── Wait After Amazon
                                  └── Comparison Generator          (Gemini Agent)
                                        └── Wait After Comparison
                                              └── Pricing & Whitespace Generator (Gemini Agent)
                                                    └── Wait After Pricing
                                                          └── Recommendations Generator     (Gemini Agent)
                                                                └── Wait
                                                                      └── Supplier Identification Generator (Mistral Agent)

[STAGE 6] Report Assembly
  HTML Assembler (Code)
    └── Clean AI Output (Code)
          └── Download HTML File (Code)
                └── See HTML Output (HTML Preview)

[STAGE 7] Google Drive Upload
  Find Wayfair Reports (Google Drive)
    └── Store Root ID (Code)
          └── Find Category Folder (Google Drive)
                └── Check Category (Code)
                      └── Category Exists? (IF)
                            ├── ✅ Exists  → Merge Category (Code)
                            └── ❌ Missing → Create Category (Google Drive) → Merge Category (Code)
                                  └── Find Date Folder (Google Drive)
                                        └── Check Date (Code)
                                              └── Date Exists? (IF)
                                                    ├── ✅ Exists  → Merge Date (Code)
                                                    └── ❌ Missing → Create Date Folder (Google Drive) → Merge Date (Code)
                                                          └── Upload to Drive
                                                               (path: Wayfair Reports / [Category] / [YYYY-MM-DD] / report.html)
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

### Stage 4 — Walmart Scraping

Walmart's anti-bot infrastructure returns a **412 Precondition Failed** response to standard HTTP requests. The workflow routes all three Walmart HTTP nodes (`Fetch Walmart Product`, `Fetch Walmart Collection`, `Fetch Walmart Product4`) using User-Agent spoofing to mimic browser traffic.

```
// Headers applied to all Walmart HTTP nodes
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
Accept: text/html
```

| Consideration | Detail |
|---|---|
| Approach | User-Agent header spoofing to mimic browser traffic |
| Latency | Standard HTTP request speed |
| Rate limiting | Existing 1.5s Wait nodes between requests |
| v2 improvement | If Walmart tightens bot detection, ScraperAPI (rotating residential proxies) is the next-level fix — treat as an infrastructure upgrade, not a workaround |

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

### Stage 5 — AI Analysis Pipeline

Six sequential AI agents, each with a focused single-job system message. **Wait nodes between every call** to respect rate limits. Four agents use Gemini; two use Mistral Large.

| Node | Model | Output |
|---|---|---|
| **Scope Generator** | Code | Research scope metadata — date, category, retailers, product counts |
| **Executive Summary Generator** | Gemini | Competitive position, top threat, top opportunity, key stats |
| **Amazon Analysis Generator** | Mistral Large | Per-retailer deep-dive: strategy label, feature advantages, Wayfair strengths, threat level |
| **Comparison Generator** | Gemini | Head-to-head feature matrix across all retailers |
| **Pricing & Whitespace Generator** | Gemini | Price band heatmap, up to 4 whitespace opportunities, risk matrix |
| **Recommendations Generator** | Gemini | Quick wins (0–30d), near-term (30–90d), strategic (90–180d), watch items |
| **Supplier Identification Generator** | Mistral Large | Brands on competitors not on Wayfair; sourcing candidates |

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
- Uses Google Drive OAuth credentials configured in n8n
- Returns a shareable link in the workflow output

> **Note:** This stage requires Google Drive OAuth credentials to be active in n8n before the final workflow run.

---

## Models & Tools

| Purpose | Tool |
|---|---|
| Executive summary, comparison, pricing & whitespace, recommendations | Google Gemini API |
| Competitor analysis (Amazon deep-dive), supplier identification | Mistral Large via Mistral Cloud API |
| Workflow automation | n8n |
| Product data scraping | HTTP Request + Cheerio (JavaScript) |
| Report delivery | Google Drive (OAuth) |
| Version control | GitHub |

**API Credentials required:**
- Google Gemini API
- Mistral Cloud API
- Google Drive OAuth

---

## Architectural Decisions & Rationale

### 1. Modular, parameterized scrapers
Each retailer scraper (Stages 2–4) uses the same parameterized HTTP + Cheerio parsing template with retailer-specific selectors injected as parameters. Adding a new competitor is a configuration change, not a rebuild — new retailer slots into the merge stage without touching existing scraper logic.

### 2. One AI node, one job
Each Stage 6 sub-node owns exactly one report section. Asking a single prompt to produce multiple sections degrades output quality across all of them. Focused prompts produce consistent, parseable JSON.

### 3. Strict JSON-only output enforced at the prompt level
Gemini defaults to wrapping output in markdown code fences (` ```json ``` `) when not explicitly constrained. This silently breaks `JSON.parse()` in downstream Code nodes. The guardrail `"Return JSON only. No markdown. No preamble."` is non-negotiable on every AI node — a lesson carried forward from Project 02.

### 4. `low_sample` flag for data quality transparency
If a retailer returns fewer than 3 products, nodes flag `low_sample: true` in the JSON output rather than silently extrapolating from insufficient data. The report assembler surfaces this as a visible caveat in the final HTML.

### 5. Wait nodes are load-bearing
Gemini rate limits will fail chained AI calls without deliberate pauses. Every Stage 6 node is preceded by a Wait node — this is not optional. Runtime increases but reliability across the full pipeline is maintained.

### 6. User-Agent spoofing as the Walmart fetch strategy
Direct HTTP requests to Walmart use browser-mimicking User-Agent headers (`Mozilla/5.0 Windows NT 10.0`) alongside standard `Accept: text/html` headers to reduce bot-detection triggers. This is the v1 approach. If Walmart tightens its TLS fingerprinting layer in future runs, ScraperAPI (rotating residential proxies + full browser rendering) is the correct infrastructure upgrade path — it should be treated as a required credential at that point, not a workaround patched in after the fact.

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
- **Bot detection is a moving target, not a one-time fix:** Walmart's 412 response can be triggered at the TLS layer depending on request patterns. User-Agent spoofing is the v1 mitigation; if detection escalates, ScraperAPI (rotating residential proxies) is the correct next step — build the escalation path in before you need it.
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
| `Wayfair_Competitor_Monitoring_Agent_Workflow.json` | n8n workflow export — all credentials stored as n8n credential references; no raw secrets |
| `.env.example` | Environment variable template — copy to `.env` and fill in values; never committed to GitHub |
| `.gitignore` | Blocks `.env`, credential exports, and OS/editor artifacts from being committed |
| `Project03_Error_Log.docx` | Error log — all bugs encountered and resolved during build |

---

*Wayfair AI Automation Externship · Extern · May 2026*
