# Wayfair AI Automation Externship

**Applied AI agent engineering for e-commerce market intelligence — built with [n8n](https://n8n.io), LLM orchestration (Mistral Large, Google Gemini), and production-grade web scraping.**

`n8n` · `JavaScript / Cheerio` · `Mistral Large` · `Google Gemini` · `HuggingFace (FLUX.1-schnell)` · `Google Drive API (OAuth)` · `ScraperAPI` · `HTML/CSS`

Repo: `lazarovaldez17/wayfair-ai-automation`

---

## What this is

This repository documents a 5-project externship (**Wayfair AI Automation, via Extern**, May 2026) built around a single question: *can a chain of automated agents take a retail category from raw web data to a decision a category manager can act on — with no human in the analysis loop?*

The category chosen for depth-of-signal was **Area Rugs** (high Amazon listing volume, active Pinterest/Instagram/blog presence, $50–$400+ price spread, and an existing Wayfair trend report available as a validation benchmark).

Each project is a standalone n8n workflow with its own README, workflow export, and sample output — but they are designed to chain together into one pipeline:

```
Project 02              Project 03              Project 04              Project 05
Market Trend         →  Competitor Monitoring →  AI Insights & Content → Dashboard Builder
Discovery                                                                 (Capstone)

"What's trending?"      "What are rivals         "What should we         "Show it to me
                          doing about it?"          do about it?"           in one view."
```

Projects 02 and 03 are the data-gathering layer. Project 04 turns that intelligence into publishable marketing content. Project 05 closes the loop by parsing the HTML outputs of 02 and 03 back out and assembling them into a single executive dashboard — no new AI reasoning, pure aggregation.

This project is aimed at a technical reader (recruiters, CS students, other engineers) who wants to see *how* it was built and *why* specific decisions were made — not just the output.

---

## Table of contents

- [Sub-projects](#sub-projects)
  - [Project 02 — Market Trend Discovery Agent](#project-02--market-trend-discovery-agent)
  - [Project 03 — Competitor Monitoring Agent](#project-03--competitor-monitoring-agent)
  - [Project 04 — AI Insights & Content Agent](#project-04--ai-insights--content-agent)
  - [Project 05 — Dashboard Builder Agent (Capstone)](#project-05--dashboard-builder-agent-capstone)
- [Engineering trade-offs](#engineering-trade-offs)
- [Tech stack](#tech-stack)
- [Repository layout](#repository-layout)
- [Skills demonstrated](#skills-demonstrated)
- [Status](#status)

---

## Sub-projects

### Project 02 — Market Trend Discovery Agent

**A 60-node n8n workflow that scrapes real product and social data to identify emerging micro-trends in a product category.**

**Problem it solves:** category teams need to know what's trending *before* it shows up in sales data. This agent scans live Amazon listings alongside social/blog trend signals and synthesizes them into a structured, consultant-style trend report.

**Pipeline:**
1. **Input & routing** — chat-triggered, validates category against an accepted list (`area_rug`, `outdoor_rug`, `hallway_runner`, `shag_rug`) before any scraping starts.
2. **Amazon scraping (dual-path)** — a collection/aisle path (search → extract product links → dedupe → paced `1.5s` waits → scrape each product page) and a direct-URL path, merged and deduplicated into one product set. Extraction uses **Cheerio** against the raw HTML.
3. **Social signal collection** — Instagram, Pinterest, blog, and market-trend feeds pulled through a third-party trend API rather than scraped directly (Instagram/Pinterest actively block bots and scraping violates their ToS).
4. **AI synthesis (Mistral Large)** — three sequential agents: classify each product as a category match, normalize inconsistent attributes (`"5x7"` / `"5'x7'"` → `5'x7'`), then identify 2–3 named micro-segments by combining product attributes with social captions and image signals.
5. **Visual generation** — converts each micro-segment into an image prompt, generates a moodboard via HuggingFace `FLUX.1-schnell`, and base64-encodes it for direct HTML embedding.

**Output:** a trend report identifying three micro-segments — *Organic Modern Neutral*, *Textured Minimalist Sculptural*, and *Bohemian Global Geometric* — each with materials, colors, target rooms, price band, and risk rating.

**Notable engineering decisions:**
- Targeting specific attribute combinations (`"8x10 modern washable"`) instead of broad category search — narrower queries return higher-signal data.
- Two independent data sources by design: Amazon captures *revealed* demand (what's already selling), social captures *aspirational* demand (what's trending before it converts to sales).
- Defensive JSON parsing — if the LLM's trend output fails to parse (e.g., wrapped in markdown fences), the workflow falls back to a default segment rather than stalling.

---

### Project 03 — Competitor Monitoring Agent

**A 65-node, 6-AI-agent n8n workflow that benchmarks Wayfair's catalog against live competitor data and produces a decision-ready competitive intelligence report.**

**Problem it solves:** manually tracking competitor pricing and feature shifts across retailers takes hours per category. This agent takes a category + competitor URLs and returns pricing gaps, feature gaps, whitespace opportunities, and sourcing recommendations in 15–20 minutes.

**Competitor selection was a strategic call, not just a technical one:**

| Retailer | Why |
|---|---|
| Amazon | Wayfair's most direct e-commerce competitor; highest SKU volume for real-time demand signal |
| Walmart | Aggressive discounting in the $27–$70 tier; direct threat on washable/functional rugs |
| Target | Threshold / Studio McGee compete directly on Wayfair's design-positioning turf ($40–$150) — the most strategically important gap an Amazon + Walmart-only analysis would miss |
| IKEA / Ruggable *(deferred)* | IKEA's real threat is in-store discovery, which no scraper captures; Ruggable is D2C with no standard product-listing path to scrape |

Target was deliberately architected as a v1 addition for its strategic overlap with Wayfair's positioning; the two retailers fully live in the current build are Amazon and Walmart.

**Pipeline:** input validation → Wayfair baseline scrape → Amazon scrape (dual-path, reused from P02) → Walmart scrape → a sequential 6-agent Gemini analysis pipeline (executive summary → competitor analysis → comparison → pricing/whitespace → recommendations → supplier ID, one agent per report section) → HTML assembly → Google Drive upload.

**Notable engineering decisions & bugs resolved:**
- **Anti-bot bypass at the right layer.** Walmart returns `412 Precondition Failed` at the TLS layer, not the application layer — rotating User-Agent headers does nothing. ScraperAPI was adopted as a required credential, not a bolt-on workaround.
- **Modular, parameterized scrapers.** Each retailer scraper reuses the same HTTP + Cheerio template with retailer-specific selectors injected as parameters, so a new competitor is a config change, not a rebuild — and the same bug surfacing in Amazon's and Walmart's scrapers (an `n8n` expression bug) was caught and fixed in both simultaneously.
- **One AI node, one report section.** Splitting the analysis into six focused Gemini calls, each owning one section, produced more consistent, parseable JSON than one prompt asked to write the whole report.
- **`low_sample` flag.** If a retailer returns fewer than 3 products, the output is flagged rather than silently treated as representative.
- **A one-character n8n bug worth knowing about:** `={{ $json.url }}` (space after the braces) resolves correctly; `=={{$json.url}}` prepends a literal `=` to every URL. A related gotcha — `null` values stringify to `"null"` inside `{{ }}` expressions, which silently passes strict-equality IF-node checks (`"null" !== ""`) — was fixed by returning empty arrays from upstream Code nodes instead of nulls.

---

### Project 04 — AI Insights & Content Agent

**Inherited a pre-built content-generation agent, audited it against real output, then rebuilt it across three improvement tracks.**

**Problem it solves:** Projects 02 and 03 answer *what's happening in the market* — this project answers the harder question, *what should Wayfair do about it*, by turning structured trend + competitor data into blog posts, Instagram captions, TikTok scripts, Pinterest pins, campaign concepts, product spotlights, and email subject lines, written in Wayfair's brand voice.

**Audit-first approach:** before changing anything, the base agent was scored against its own sample output. It produced a clean 8-section report structure but fell short in three specific ways — which became the three tracks, implemented in priority order because each compounds the next:

| Track | Problem found | Fix |
|---|---|---|
| **3 — Data inputs** *(fixed first — root cause)* | Prompt schema shipped hardcoded example metrics (`28%`, `$127–$156`) that the model mirrored instead of replacing; all 3 of P02's micro-segments collapsed into one generic theme | Stripped example values from the schema; built a `Parse P03 Gaps` node to extract real competitor pricing/whitespace before the AI runs; passed all 3 P02 segments as explicit structured inputs |
| **1 — Voice** *(fixed second — most visible impact)* | Output opened with statistics and described products before evoking any feeling — the inverse of Wayfair's room-first brand voice | Audited Wayfair's actual blog/Instagram copy, added a brand-voice reference block plus five explicit **negative constraints** (no `"Did you know"`, no stat-first openers), and split the prompt into 5 channel-specific sub-prompts |
| **2 — Content types** *(fixed third)* | Agent produced content *briefs* (a title and a one-line description) rather than usable drafts | Added a `draft_opening` field (150–200 words) to blog ideas, a structured 4-field TikTok script block, full Pinterest descriptions, and per-segment product spotlights |

**Reliability guardrails worth calling out:**
- **Negative constraints outperformed positive-only examples** for eliminating default LLM patterns — telling the model what *not* to do worked more reliably than showing it good examples alone.
- **`"never invent values"` instructions reduce but don't eliminate hallucination** — a code-level validation pass that checks generated metrics against the source text was added as a second layer of defense.
- **3-layer fallback for critical data** (node reference → inline extraction → hardcoded real value) so the report degrades gracefully instead of breaking silently when upstream data is missing.
- **LangChain agent nodes don't pass upstream data through their output** — data that needs to survive a LangChain hop needs its own resolution path downstream. (Undocumented behavior, found the hard way.)

**Evaluated result:** scored 8.2/10 across content depth, data integrity, strategic value, and structural completeness after all three tracks — the report's own conclusion is "conditionally production-ready," with one human review pass recommended before publishing generated percentage figures.

---

### Project 05 — Dashboard Builder Agent *(Capstone)*

**Aggregates the outputs of Projects 02 and 03 into a single executive dashboard — deliberately without any new AI reasoning.**

**Problem it solves:** answering "what's trending, how do we compare on price, where are the gaps, what should we prioritize" today means opening three separate HTML reports and synthesizing them by hand. This agent does it in under a minute.

**Pipeline:** OAuth into Google Drive → locate the latest P02 and P03 HTML reports by category and date → parse both with **Cheerio** using known CSS selectors (`.segment-name`, `.price-band`, `.gap-row`, `.whitespace-item`, …) → assemble a static HTML dashboard via string interpolation → validate structure → download / optionally re-upload to Drive.

**The central engineering decision of this project — parsing over AI reasoning:**

| | Cheerio parsing | AI re-reading the reports |
|---|---|---|
| Speed | Milliseconds | 10–30s per call |
| Cost | Free | API tokens consumed |
| Reliability | 100% deterministic | Output can vary run to run |
| Used in P05? | ✅ | ❌ |

The upstream agents already did the reasoning work; asking an LLM to re-read and re-summarize their HTML output would be slower, more expensive, and less consistent than just reading the structured HTML directly. This is the one project in the pipeline that argues *against* reaching for AI.

**Delivered output:** a single self-contained `Area_Rug_Dashboard.html`, built from a live 68-product sample (10 Amazon, 10 Walmart, 48 Wayfair), organized into 6 navigable views:

| View | Surfaces |
|---|---|
| Executive Overview | Category health snapshot, target micro-segment, key trend + competitive insight, top alerts, top 3 actions |
| Market & Trends | Market size/CAGR, 3 trending micro-segments with palettes and target rooms, moodboard |
| Competitive Intel | Wayfair vs. Amazon vs. Walmart price positioning, side-by-side attribute comparison, per-retailer profiles |
| Opportunity Radar | Price-band heatmap, ranked whitespace opportunities (e.g., no Wayfair SKUs under $100 despite active Walmart demand there), supplier gaps |
| Risk & Diagnostics | 5 categorized risks (sustainability, trend obsolescence, performance, price, care) each with a severity rating and a mitigation recommendation |
| Action Center | Quick wins, strategic initiatives, and an owner/due-date action tracker |

**Status:** complete. This closes the pipeline exactly as designed — demonstrating that four separately-built agents can be wired into one coherent internal tool. The value here is integration, not novel AI logic: every number on the dashboard was reasoned about once, upstream, and the capstone's only job is to surface it fast.

---

## Engineering trade-offs

Patterns that recur across all four projects, synthesized:

**Parsing vs. AI reasoning.** Used deterministic parsing (Cheerio) wherever the data already exists in a known, structured location (P05's entire premise), and reserved LLM calls for genuine synthesis and generation tasks (trend identification in P02, content writing in P04). Treating every extraction problem as a prompting problem is slower and less reliable than it looks.

**Reliability engineering for LLM outputs, in layers.** Strict "JSON only, no markdown, no preamble" prompt guardrails; explicit negative constraints (more effective than positive examples alone at killing default LLM patterns like "Did you know…"); a code-level post-generation validation pass to catch hallucinated metrics that prompt instructions alone didn't stop; and a 3-layer fallback chain for any data path a report depends on, so failures degrade gracefully instead of breaking the whole output.

**Modular, parameterized scraping over one-off scripts.** Every retailer scraper across P02/P03 is the same HTTP + Cheerio template with retailer-specific selectors injected as parameters. New retailer = configuration change, not new code — and the trade-off paid off directly when the same expression bug appeared in both the Amazon and Walmart scrapers and was fixed once, in both places, instead of twice, separately.

**Infrastructure-level fixes over incremental workarounds.** When Walmart's anti-bot layer turned out to be TLS-level (not header-level) detection, the fix was adopting ScraperAPI as a first-class credential rather than iterating on User-Agent rotation that could never have worked.

**Sequencing improvements by leverage.** In Project 04, data-input fixes were implemented before voice or content-type work, on the reasoning that a well-voiced Instagram caption written about the wrong product segment is still wrong — fix what everything downstream depends on first.

**Data-quality transparency over silent extrapolation.** Both P02 and P03 flag `low_sample` conditions in their JSON output rather than quietly treating a 2-product sample the same as a 50-product one.

**Evaluation as part of the build, not an afterthought.** Project 04's enhancement tracks were each scored against real generated output before moving to the next — that scoring pass, not code review, is what surfaced the LangChain data-loss bug.

---

## Tech stack

| Purpose | Tool |
|---|---|
| Workflow automation / orchestration | [n8n](https://n8n.io) |
| Content generation, trend synthesis, normalization | Mistral Large (Mistral Cloud API) |
| Competitive analysis & report writing | Google Gemini API |
| Structured HTML parsing / extraction | Cheerio (JavaScript) |
| Moodboard image generation | HuggingFace — `FLUX.1-schnell` |
| Product data scraping | HTTP Request nodes + Cheerio |
| Anti-bot scraping infrastructure | ScraperAPI |
| Secure file retrieval | Google Drive API (OAuth) |
| Social/blog trend signals | Extern trend API |
| Report & dashboard templating | HTML / CSS (hand-built, no external UI libraries) |
| Version control | GitHub |

---

## Repository layout

Each project is self-contained with its own README, workflow export, and sample output artifacts:

```
wayfair-ai-automation/
├── README.md                              ← you are here
├── project02-market-trend-discovery/
│   ├── README.md
│   └── workflow.json                      ← n8n export, 60 nodes
├── project03-competitor-monitoring/
│   ├── README.md
│   ├── Competitor_Analysis_Report.html    ← final deliverable
│   ├── Competitor_Analysis_Report.pdf
│   ├── Wayfair_Competitor_Monitoring_Agent_Workflow_SAFE.json  ← credentials stripped
│   ├── Project03_Error_Log.docx
│   ├── .env.example
│   └── .gitignore                         ← blocks .env / credential exports
├── project04-content-strategy-agent/
│   ├── README.md
│   └── Complete_Content_Strategy_Agent.json  ← v7.2, all tracks + fixes applied
└── project05-dashboard-builder/            ← capstone
    ├── README.md
    └── Area_Rug_Dashboard.html             ← final deliverable, 6-view unified dashboard
```

Credential handling follows the same pattern throughout: workflow exports have session cookies and API keys stripped and replaced with environment-variable references, `.env.example` documents what's required without shipping real values, and `.gitignore` blocks credential files from ever being committed.

---

## Skills demonstrated

- **Workflow orchestration** — multi-stage, branching n8n pipelines (60–65 nodes) with conditional routing, error paths, and rate-limit-aware sequencing
- **Web scraping at scale** — dual-path scraping strategies, anti-bot evasion (TLS-layer detection bypass), modular per-retailer scraper design
- **LLM/prompt engineering** — structured JSON output enforcement, negative-constraint prompting, channel-specific sub-prompting, multi-layer hallucination defense
- **Multi-agent pipeline design** — sequential specialized agents (one job per agent) across two different model providers (Mistral, Gemini)
- **API integration & OAuth** — Google Drive delegated access, third-party trend APIs
- **Structured data extraction** — Cheerio/CSS-selector-based parsing as a deliberate alternative to AI re-processing
- **Front-end templating** — hand-built responsive HTML/CSS report and dashboard generation, no framework dependency
- **Evaluation-driven iteration** — scoring real output against a rubric after each change, rather than shipping on code review alone
- **Systems integration** — designing a capstone whose entire value is connecting four independently-built agents into one coherent tool

---

## Status

| Project | Status |
|---|---|
| P02 — Market Trend Discovery | Core pipeline (stages 1–3) built; visual generation (stage 4) in progress |
| P03 — Competitor Monitoring | Complete — all 8 stages finalized, report delivered |
| P04 — AI Insights & Content | Complete — all 3 enhancement tracks implemented and evaluated |
| P05 — Dashboard Builder | Complete — capstone dashboard delivered (`Area_Rug_Dashboard.html`) |

---

*Wayfair AI Automation Externship · via Extern · May 2026*
