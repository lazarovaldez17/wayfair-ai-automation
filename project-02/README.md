# Project 02: Market Trend Discovery AI Agent

**Externship:** Wayfair AI Automation via Extern
**Category Focus:** Area Rugs
**Workflow:** Market Trend Discovery Agent (116 nodes — 90 functional + 26 documentation sticky notes)
**Status:** ✅ Complete — end-to-end pipeline shipped

\---

## Overview

An automated market intelligence agent that scans Amazon product listings, Instagram, Pinterest, industry blogs, and market data to identify emerging trends in the Area Rug category. The agent runs a multi-agent AI pipeline (Mistral for data processing, Gemini for report writing, HuggingFace FLUX for visuals) and outputs a professionally styled, downloadable HTML trend report — micro-segment analysis, risk ratings, moodboard visuals, and actionable merchandising recommendations included.

\---

## Why Area Rugs?

|Factor|Rationale|
|-|-|
|Dataset richness|High Amazon listing volume with diverse attributes|
|Social signal volume|Active presence on Pinterest, Instagram, and design blogs|
|Price diversity|Products spanning $50–$400+, enabling multi-tier analysis|
|Benchmark availability|Wayfair Area Rug Trend Report PDF available for output validation|

\---

## Sample Output: What the Agent Produces

Two trend reports were generated to demonstrate the agent's range — one broad category scan, one narrow sub-category dive — plus moodboard images for the top micro-segments.

### Report 1 — Area Rug Trend Report (broad lens)

📄 `project02\_Area\_Rug\_Trend\_Report.pdf`

A full Area Rug category scan that mirrors the structure of Wayfair's internal trend report.

* **Executive insight:** Modern area rugs are shifting away from flat geometric abstracts toward sculptural, tactile, biophilic forms in neutral palettes — a premium-positioning opportunity for Wayfair.
* **Top-performing attributes:** 8'×10' size, modern/abstract patterns, beige and multi-color palettes, polyester/synthetic materials, $100–$200 sweet spot.
* **Three identified micro-segments:**

  * *Organic Modern Neutral* (premium) — soft biophilic shapes, warm sand/cream tones
  * *Textured Minimalist Sculptural* (mid-to-premium) — high-low carved textures, monochromatic palettes
  * *Bohemian Global Geometric* (mid-range) — vibrant tribal patterns, earthy artisan tones
* **Risk ratings:** Sustainability = HIGH (polyester dominance), Trend Obsolescence = HIGH, Performance and Price = MEDIUM, Care = LOW — each paired with mitigation strategies (rPET sourcing, OEKO-TEX badging, rug-pad cross-sell).
* **Strategic output:** Five prioritized recommendations spanning assortment, messaging, pricing, UX tools, and marketing focus, plus quick wins and a watch list (sustainability demand, smart-rug tech, blended aesthetics).

### Report 2 — Machine-Washable Area Rugs Trend Report (narrow lens)

📄 `project02\_Specific\_Area\_Rug\_Trend\_Report.pdf`

A focused 2026–2027 forward look at a specific sub-category: machine-washable rugs for family and pet households.

* **Market sizing:** Global washable rug sales \~$6.2–6.5B in 2024, projected to double by 2033 at a 6.8–9.2% CAGR — outpacing the broader rug market (\~4–6%).
* **Demand signal:** "Washable rug" search interest +47% year-over-year (\~270K monthly searches), with Instagram identified as the key discovery channel.
* **Two style micro-segments:**

  * *Vintage/Distressed Medallion Washable* — Persian-inspired antique look in greiges and warm taupes (the "quiet luxury" play)
  * *Modern Braided/Geometric Washable* — concentric/braided textures, 70s-inspired warm tones restrained for "texture maximalism"
* **Material profile:** Polyester (including recycled PET) as the dominant base fiber; microfiber/chenille for soft-hand bedroom rugs; performance blends (Ruggable-style two-piece systems).
* **Pain points surfaced:** Thin constructions, large sizes that don't fit standard washers, curling edges and floor-damage concerns from certain pad systems.

### AI-Generated Moodboards

Each micro-segment was rendered as a photorealistic interior moodboard via the FLUX.1-schnell pipeline (Mistral prompt → cleaning → HuggingFace inference → base64 embed in the final report).

* 🖼️ `Area\_Rug\_AI\_Generated\_image.png` — visualizes the *Organic Modern Neutral* segment: cream/sand wavy rug with biophilic high-low texture in a sun-washed minimalist living room.
* 🖼️ `AI\_Generated\_Image\_Area\_Rug.png` — visualizes the *Bohemian Global Geometric* segment with mid-century styling cues: bold teal/mustard/orange circle pattern in a 70s-inspired interior.

\---

## Workflow Architecture (8 Steps)

```
\[STEP 1] Input \& Routing
  Chat Trigger
    └── Input Parser1 (Code)
          └── Category Validator (IF)
                ├── ✅ Valid  → 3 parallel branches into Step 2A + 2B
                └── ❌ Invalid → Error Response

\[STEP 2A] Amazon Scraping — Dual Path
  Path 1 — Collection/Aisle:
    Prepare Collection URLs → Is List Empty? (IF)
      └── Visit Aisle Page (HTTP) → Extract Product Links (HTML)
            → Remove Duplicate Links → Pause for Safety (Wait 1.5s)
              → HTTP Request (product page) → Read Item Details (Cheerio)

  Path 2 — Direct URLs:
    Prepare Direct Links → Do We Have Direct Links? (IF)
      └── Visit Direct Product Page (HTTP) → Read Direct Item Details

  Merge Amazon Products (Collection + Direct)

\[STEP 2B] Extern API — Social, Blogs, Market
  Get Category Key (Code)
    ├── Fetch Instagram (HTTP)
    ├── Fetch Pinterest  (HTTP)
    ├── Fetch Blogs      (HTTP)
    └── Fetch Market Data (HTTP)
  → Merge Social Data → Merge Image Analysis → Merge All Data

\[STEP 3] AI Processing — Mistral (4 agents)
  Wait → Judge Product Category (Mistral) → keeps "match" / drops "no\_match"
    → Remove Invalid Products
      → Standardize Details (Mistral) → normalize size, color, material,
         pattern, style, pile, price
        → Save Standardized Data
          → Identify Top Trends (Mistral) → ≥1 micro-segment per category
            → Save Trend List → Split Segments for Images

\[STEP 4] Image Generation (per micro-segment loop)
  Wait → Generate Image Prompt (Mistral, <150 words, single paragraph)
    → Clean Prompt (Code) → strip markdown, normalize quotes
      → Generate Image (HuggingFace FLUX.1-schnell, 120s timeout)
        → Encode to Base64 (data:image/png;base64,…)
          → Collect All Images

\[STEP 5–6] Multi-Agent Report Writing — Gemini (8 agents)
  Blog Knowledge Extractor (Gemini) → Parse Blog Knowledge
  Scope Generator (Code, static metadata)
  → run in sequence with Wait gates between:
      ├── Executive Summary Generator       (Gemini)
      ├── Market Research Generator         (Gemini)
      ├── Category Generator                (Gemini)
      ├── Attribute Generator               (Gemini)
      ├── Visual Trends Generator           (Gemini, image-aware)
      ├── Risks Generator                   (Gemini)
      └── Recommendations Generator         (Gemini)

\[STEP 7] Final Output
  Collect All Sections
    → Assemble HTML Report (inline CSS, base64 images embedded)
      → Section Validator (regex check: Exec Summary, Scope, Category,
         Attribute, Risk all present)
        → Create Download File (timestamped .html)
          → HTML Preview (in-app render)

\[STEP 8] Google Drive Upload
  Find Wayfair Reports (root folder search)
    → Store Root ID
      → Find Category Folder (e.g. "area-rug")
        → Category Exists? (IF)
            ├── ✅ Exists  → Merge Category
            └── ❌ Missing → Create Category → Merge Category
              → Find Date Folder (YYYY-MM-DD)
                → Date Exists? (IF)
                    ├── ✅ Exists  → Merge Date
                    └── ❌ Missing → Create Date Folder → Merge Date
                      → Wait1
                        → Upload to Drive
                           (path: Wayfair Reports / \[Category] / \[YYYY-MM-DD] / report.html)
```

\---

## Models \& Tools

|Layer|Tool|Role|
|-|-|-|
|Data agents (4)|**Mistral Large** via Mistral Cloud|Product classification, attribute normalization, micro-segment identification, image prompt drafting|
|Writer agents (8)|**Google Gemini**|Blog knowledge extraction + 7 specialist section writers (Executive Summary, Market Research, Category, Attribute, Visual Trends, Risks, Recommendations)|
|Image generation|**HuggingFace — FLUX.1-schnell**|Moodboard visuals (one per micro-segment, base64-embedded in report)|
|Sales data|Amazon (HTTP + Cheerio)|Product attributes, pricing, ratings|
|Social/blog/market signals|**Extern API** (`xx.xxx.xxx.xxx:xxxx`)|Instagram, Pinterest, blogs, market data endpoints|
|Workflow automation|**n8n**|116-node orchestration|
|Persistence|**Google Drive**|Final HTML report stored under `Wayfair Reports / \[Category] / \[YYYY-MM-DD] /` for downstream use by Project 05 Dashboard Builder|
|Version control|**GitHub**|Portfolio repo|

\---

## Node Type Distribution

|Type|Count|Purpose|
|-|-|-|
|Code (JS)|29|Data shaping, parsing, dedup, HTML assembly, validation, Drive folder logic|
|Sticky Note|26|In-canvas documentation per step|
|LC Agent|12|4 Mistral data + 8 Gemini writer agents|
|Wait|11|Rate-limit pauses between chained AI calls and Drive upload|
|HTTP Request|8|Amazon scraping + Extern API endpoints|
|Gemini Chat Model|8|One per writer agent|
|Google Drive|6|Folder search, folder creation, and HTML report upload (Stage 8)|
|IF|6|Conditional routing (category validation, branch selection, Drive folder existence checks)|
|Mistral Chat Model|4|One per data agent|
|Merge|3|Dual-path joins (Amazon, social, full dataset)|
|HTML|2|Cheerio extraction + final report preview|
|Chat Trigger|1|Entry point|
|**Total**|**116**||

\---

## Key Learnings

* **Aisle logic > broad search:** Specific attribute combos (e.g., "8x10 modern washable") return higher-signal data than generic category searches
* **Dual data sources required:** Amazon captures revealed demand (what's selling); Instagram/Pinterest/blogs capture aspirational demand (what's trending before it sells)
* **Normalize before analyzing:** Raw product data is inconsistent — standardizing attributes first dramatically improves micro-segment accuracy
* **Wait nodes are infrastructure, not decoration:** Mistral Cloud and Gemini both rate-limit chained calls; without `Wait` nodes the workflow stalls on 429s
* **Fallback JSON parsing is mandatory:** AI outputs include markdown code fences (````json ... ````) — every parser node strips them before `JSON.parse()` and falls back to a default object on failure
* **Split data tasks from writing tasks across models:** Mistral handles deterministic classification/extraction (cheaper, faster), Gemini handles narrative section writing (better prose quality, longer context)
* **One writer agent per section beats one monolithic writer:** Easier to debug, regenerates faster on failure, lets each agent over-specialize on its section's tone

\---

## Resolved Improvements (from previous README)

* ✅ **HTML report output** — replaced inline chatbox responses with `Assemble HTML Report` → `Section Validator` → `Create Download File` pipeline
* ✅ **Fallback JSON parsing** — every code node that parses AI output strips markdown fences and returns a default object on parse failure
* ✅ **Polyester sustainability flag** — surfaced in the Risks Generator (HIGH RISK rating + rPET / OEKO-TEX mitigation guidance)
* ✅ **Section presence validation** — regex check confirms Executive Summary, Scope, Category, Attribute, and Risk sections all rendered before the file is created
* ✅ **Google Drive persistence** — replaced Supabase with a 14-node Stage 8 pipeline that auto-creates `Wayfair Reports / \[Category] / \[YYYY-MM-DD] /` folder structure and uploads the final HTML report; output is consumed directly by the Project 05 Dashboard Builder Agent

\---

## Open Improvements

* \[ ] Make the Extern API endpoint configurable via env var rather than hardcoded
* \[ ] Add a retry-with-backoff wrapper around the HuggingFace call for cold-start timeouts
* \[ ] Add structured run logging (node-level error capture + segment/category combo tracking) to a persistent store for post-run analysis

\---

## Files

|File|Description|
|-|-|
|`README.md`|Project documentation (this file)|
|`Market\_Trend\_Discovery\_Agent.json`|Exported n8n workflow — Market Trend Discovery Agent (116 nodes)|
|`Area\_Rug\_AI\_Generated\_image.png`|AI-generated moodboard — Organic Modern Neutral micro-segment|
|`AI\_Generated\_Image\_Area\_Rug.png`|AI-generated moodboard — Bohemian Global Geometric micro-segment|
|`Project02\_Error\_Log.docx`|Build log — errors encountered, root causes, and fixes|

\---

*Wayfair AI Automation Externship · Extern · May 2026*

