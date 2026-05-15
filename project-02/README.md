# Project 02: Market Trend Discovery AI Agent

**Externship:** Wayfair AI Automation via Extern
**Category Focus:** Area Rugs
**Workflow:** Market Trend Discovery Agent (95 nodes)
**Status:** Stages 1–5 Complete · Report Generation In Progress

---

## Overview

An automated market intelligence agent that scans real-world data sources — Amazon product listings, Instagram, Pinterest, and industry blogs — to identify emerging trends in the Area Rug category. The agent outputs a structured, consultant-style HTML trend report with micro-segment analysis, AI-generated moodboard images, risk ratings, and actionable merchandising recommendations.

---

## Why Area Rugs?

| Factor | Rationale |
|---|---|
| Dataset richness | High Amazon listing volume with diverse attributes |
| Social signal volume | Active presence on Pinterest, Instagram, and design blogs |
| Price diversity | Products spanning $50–$400+, enabling multi-tier analysis |
| Benchmark availability | Wayfair Area Rug Trend Report PDF available for output validation |

---

## Full Workflow Architecture (95 Nodes)

```
[STAGE 1] Input & Routing
  Chat Trigger
    └── Input Parser (Code)
          └── Category Validator (IF)
                ├── ✅ Valid → 3 parallel paths
                └── ❌ Invalid → Error Response

[STAGE 2A] Amazon Scraping — Dual Path
  Path 1 (Collection/Aisle):
    Prepare Collection URLs → Is List Empty? (IF)
      └── Visit Aisle Page (HTTP) → Extract Product Links (HTML)
            → Remove Duplicate Links → Pause for Safety (1.5s Wait)
              → Visit Product Page (HTTP) → Read Item Details (Cheerio)

  Path 2 (Direct URLs):
    Prepare Direct Links → Do We Have Direct Links? (IF)
      └── Visit Direct Product Page (HTTP) → Read Direct Item Details

  Both paths → Merge Amazon Products → Deduplicate + Combine Final List

[STAGE 2B] Social & Market Signals (Extern API)
  Get Category Key
    ├── Fetch Instagram   → /api/trends/pinterest  (limit: 20)
    ├── Fetch Pinterest   → /api/trends/pinterest  (limit: 20)
    ├── Fetch Blogs       → /api/trends/blogs      (limit: 10)
    └── Fetch Market Data → /api/trends/blogs      (limit: 10, type: market)
          └── Merge Social Data → Merge Social Data1 → Merge Image Analysis

  All data → Merge All Data 1 → Wait2

[STAGE 3] AI Synthesis (Mistral Large)
  Judge Product Category (Agent)     → classify: match / no_match / uncertain
    └── Remove Invalid Products (Code)
          └── Standardize Details (Agent)   → normalize size, color, material, style
                └── Save Standardized Data (Code)
                      └── Wait → Identify Top Trends (Agent) → 2–3 micro-segments
                            └── Save Trend List (Code)
                                  └── Split Segments for Images → IF check

[STAGE 4] Visual Generation (Mistral + HuggingFace)
  Wait3 → Generate Image Prompt (Mistral Agent)
    └── Clean Prompt (Code)
          └── Generate Image — FLUX.1-schnell (HuggingFace HTTP)
                └── Encode to Base64 (Code)
                      └── Collect All Images (Code) ← aggregates all loop iterations

[STAGE 5] Report Generation (Google Gemini)
  Blog Knowledge Extractor (Gemini Agent) → Parse Blog Knowledge (Code)
    └── Scope Generator (Code)
          └── Wait4 → Executive Summary Generator (Gemini Agent)
                └── Wait5 → Market Research Generator (Gemini Agent)
                      └── Category Generator (Gemini Agent)
                            └── Wait6 → Attribute Generator (Gemini Agent)
                                  └── Wait7 → Visual Trends Generator (Gemini Agent)
                                        └── Wait8 → Risks Generator (Gemini Agent)
                                              └── Wait9 → Recommendations Generator (Gemini Agent)
```

---

## Stage-by-Stage Breakdown

### Stage 1 — Input & Routing

- **Trigger:** Chat message containing a category name + Amazon URLs
- **Input Parser:** Extracts `selectedCategory`, `categoryKey`, `collectionUrls[]`, `directUrls[]`, and `isValid` flag
- **Category Validator:** IF node checks `isValid === true` → branches to continue or error path
- **Accepted categories:** `area_rug`, `outdoor_rug`, `hallway_runner`, `shag_rug`

**Test message format:**
```
Area Rug - https://www.amazon.com/s?k=8x10+modern+washable+area+rug https://www.amazon.com/s?k=organic+modern+neutral+area+rug https://www.amazon.com/dp/[ASIN1] https://www.amazon.com/dp/[ASIN2]
```

---

### Stage 2A — Amazon Scraping (Dual-Path)

**Collection Path:**
1. Splits collection URLs into individual items
2. Guards against empty URLs with IF check
3. Visits aisle/search page with browser headers (anti-bot disguise)
4. Extracts product links via CSS selectors
5. Removes duplicates, keeps top results
6. Applies **1.5s Wait** between requests to avoid rate limiting
7. Visits each product page and extracts: title, price, material, size, pattern, rating, review count via **Cheerio** parser

**Direct Path:**
1. Splits direct product URLs into individual items
2. Guards against empty entries with IF check
3. Visits and scrapes each page immediately (same Cheerio extraction logic)

**Merge:** Both paths combine → deduplicate by URL → single `amazonProducts` array with `totalProducts` count

---

### Stage 2B — Social Trend Signals (Extern API)

| Node | Endpoint | Limit |
|---|---|---|
| Fetch Instagram | `/api/trends/pinterest` | 20 items |
| Fetch Pinterest | `/api/trends/pinterest` | 20 items |
| Fetch Blogs | `/api/trends/blogs` | 10 items |
| Fetch Market Data | `/api/trends/blogs` (market type) | 10 items |

All four feeds merge → cleaned → passed into the AI synthesis stage alongside Amazon data.

**Why Extern API instead of direct scraping:** Instagram and Pinterest block bots, violate ToS on direct scraping, and require login/proxies. Extern provides pre-cleaned, regularly updated social trend data via standard GET requests.

---

### Stage 3 — AI Synthesis (Mistral Large)

Three sequential AI agents with **Wait nodes** between calls to avoid rate limits:

| Agent | Task | Output |
|---|---|---|
| **Judge Product Category** | Labels each Amazon product: `match`, `no_match`, or `uncertain` against the selected category | Filtered product set |
| **Standardize Details** | Normalizes attributes — e.g., `"5x7"` / `"5'x7'"` / `"5 ft by 7 ft"` → `5'x7'`; `"Grey"` / `"Charcoal Grey"` → `Gray` | Consistent product JSON |
| **Identify Top Trends** | Combines product attributes + social captions/hashtags + visual image analysis → identifies 2–3 named micro-segments | Segment definitions with colors, materials, rooms, image keywords |

**Fallback logic in Save Trend List:** If Mistral's JSON output fails to parse (e.g., markdown code fences), defaults to a `Classic Neutral` segment so the workflow never stalls.

**Identified micro-segments (Area Rugs):**
1. **Organic Modern Neutral** — soft biophilic shapes, warm neutrals, premium positioning
2. **Textured Minimalist Sculptural** — high-low carved textures, monochromatic palettes, mid-to-premium
3. **Bohemian Global Geometric** — vibrant tribal patterns, earthy artisan tones, mid-range

---

### Stage 4 — Visual Generation (Mistral + HuggingFace)

- **Generate Image Prompt (Mistral):** Converts each micro-segment's attributes into a photorealistic interior design prompt (<150 words, single paragraph, no markdown)
- **Clean Prompt (Code):** Strips line breaks, markdown, replaces double quotes — formats for HuggingFace API
- **Generate Image (HuggingFace):** Calls `FLUX.1-schnell` model → returns PNG binary (120s timeout)
- **Encode to Base64 (Code):** Converts PNG to `data:image/png;base64,...` for direct HTML embedding
- **Collect All Images (Code):** Aggregates all per-segment loop iterations back into a single item using `$input.all()`

> ⚠️ **Note on execution context:** `Collect All Images` resets the n8n execution context for all downstream nodes. Pre-loop node data (e.g., `Save Standardized Data`, `Save Trend List`) must be forwarded through the first post-loop Code node (`Scope Generator`) rather than referenced directly in AI Agent prompt expressions. See [Error Log](./Project02_Error_Log.docx) — ERR-001.

---

### Stage 5 — Report Generation (Google Gemini)

Eight sequential Gemini AI agents, each generating one HTML section of the final trend report:

| Agent | Report Section | Key Inputs |
|---|---|---|
| **Blog Knowledge Extractor** | Blog material/care/sizing insights | Extern blog feed |
| **Scope Generator** *(Code)* | Research scope card grid | Social counts, product count — also forwards pre-loop data to all downstream agents |
| **Executive Summary Generator** | Key insight, findings, why it matters | Attribute summary, micro-segments, blog insights |
| **Market Research Generator** | Market size, CAGR, segments, buyer profile | Price distribution, materials summary |
| **Category Generator** | Category definition, sizing guide | Blog knowledge, category key |
| **Attribute Generator** | Materials, sizes, colors, price analysis | Full standardized product data |
| **Visual Trends Generator** | Visual story, dominant colors, moodboards | Image analysis, generated moodboard images |
| **Risks Generator** | Risk assessment with mitigation strategies | Top materials, sustainability signals |
| **Recommendations Generator** | Strategic recommendations, quick wins, watch list | Full attribute summary, micro-segments |

**Wait nodes** between each agent prevent Gemini API rate limit errors (503s).

---

## Models & Tools

| Purpose | Tool |
|---|---|
| Product classification, attribute standardization, trend analysis, image prompts | Mistral Large (Mistral Cloud API) |
| Report section generation (8 agents) | Google Gemini API |
| Moodboard image generation | HuggingFace — FLUX.1-schnell |
| Social/blog trend data | Extern API (`34.196.186.128:8000`) |
| Workflow automation | n8n |
| Version control | GitHub |

---

## Key Learnings

- **Aisle logic > broad search:** Targeting specific attribute combos (e.g., "8x10 modern washable") returns higher-signal data than generic category searches
- **Dual data sources required:** Amazon captures revealed demand (what's selling); social signals capture aspirational demand (what's trending before it sells)
- **Normalization before analysis:** Raw product data is inconsistent — standardizing first dramatically improves micro-segment accuracy
- **Wait nodes are essential:** Both Mistral Cloud and Gemini API rate limits require deliberate pauses between chained AI agent calls
- **Fallback JSON parsing:** AI output can include markdown code fences — stripping them before `JSON.parse()` prevents workflow stalls
- **n8n execution context resets after aggregation:** After any `$input.all()` aggregation, downstream AI Agent nodes cannot reference pre-loop nodes via `$()` expressions. Forward required data through the first post-loop Code node instead

---

## Errors Encountered & Resolved

| ID | Error | Node | Status |
|---|---|---|---|
| ERR-001 | n8n Execution Context Break — 'Node Not Found' After Loop Aggregation | Executive Summary Generator | ✅ Resolved |
| ERR-002 | Gemini API 2.5 Flash — 503 Service Unavailable | Report Generation Agents | ✅ Resolved |

Full details in [`Project02_Error_Log.docx`](./Project02_Error_Log.docx).

---

## Self-Identified Improvements

- [x] ~~Add validation + fallback logic for missing product fields (price, size) — retry parsing or flag for review~~ → Addressed via `Save Standardized Data` fallback parsing and `Save Trend List` default segment logic
- [x] ~~Restructure final output to generate a formatted HTML/PDF report file rather than returning results inline in the n8n chatbox~~ → Implemented via Stage 5 Gemini report generation pipeline
- [ ] Add automated sustainability risk flag: if dominant material is Polyester → trigger HIGH RISK warning and suggest rPET or OEKO-TEX alternatives
- [ ] Enable "Retry On Fail" (3–5 attempts, 2–5s delay) on all Gemini AI Agent nodes to handle intermittent 503 errors automatically

---

## Files

| File | Description |
|---|---|
| `README.md` | Project documentation (this file) |
| `workflow.json` | Exported n8n workflow — Market Trend Discovery Agent (95 nodes) |
| `Project02_Error_Log.docx` | Debugging journal — errors encountered, root causes, and fixes applied |

---

*Wayfair AI Automation Externship · Extern · May 2026*
