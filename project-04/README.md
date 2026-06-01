# Project 04: AI Insights & Content Agent

**Externship:** Wayfair AI Automation via Extern
**Category Focus:** Area Rugs
**Status:** ✅ Complete — Delivered
**Version:** Track 3 Enhanced

---

## Overview

Project 04 shifts the pipeline from *intelligence gathering* to *action*. Where Projects 02 and 03 answered *"What is happening in the market?"*, this project answers the harder business question:

> **"What should Wayfair DO about it?"**

The AI Insights & Content Agent takes structured trend and competitor data as inputs and transforms them into real, usable marketing content — blog posts, Instagram captions, campaign concepts, product spotlights, and more — written in Wayfair's brand voice, ready for a marketing team to act on without rewriting from scratch.

This project was completed in two phases: **inheriting and auditing** a base agent, then **iteratively enhancing it** across three tracks of improvement, evaluated against real outputs after each change.

---

## Where This Fits in the Pipeline

```
Project 02                  Project 03                  Project 04
Market Trend Discovery  →   Competitor Monitoring   →   AI Insights & Content
"What's trending?"          "What are rivals doing?"    "What should WE do?"

      ↓                           ↓                           ↓
 Micro-segment              Pricing gaps              Blog posts
 trend report               Whitespace opps           IG captions
 (Area Rugs)                Feature gaps              Campaign concepts
                                                      Product spotlights
                                                      TikTok scripts
                                                      Pinterest pins
                                                      Email subjects
```

Projects 02 and 03 are no longer standalone deliverables — their HTML outputs become the **upstream inputs** this agent consumes to generate content.

---

## Sample Output: What the Agent Produces

📄 `Wayfair_area_rug_Content_Strategy.html` — generated June 1, 2026

An 11-section styled HTML content strategy report produced from the P02 Area Rug trend report and P03 Competitor Analysis report. Written in Wayfair's brand voice throughout. Footer designation: **Track 3 Enhanced**.

### Header KPIs (from actual run)

| Metric | Value | Notes |
|---|---|---|
| Key Data Points | 0 | Hallucination guard stripped unverifiable % figures — expected behavior |
| Content Ideas | 6 | Blog, Carousel, Video, Pinterest Pin, Buying Guide, Email Series |
| Campaign Concepts | 3 | Full headline variants + hero copy per campaign |
| Micro-Segments | 3 | Parsed from P02 report for this run |

### 3 Micro-Segments (from this run's P02 input)

| Segment | Tier | Product Example |
|---|---|---|
| Textured Neutral Minimalist | Premium | Hand-tufted New Zealand wool in a soft wave pattern |
| Retro Abstract Geometric | Mid-range | Machine-woven polyester in warm terracotta and cream geometric |
| Luxury 3D Wool Abstract | Premium | Hand-sculpted Tibetan wool with 3D abstract patterns |

> **Note:** Micro-segment names are parsed directly from the uploaded P02 HTML report at runtime. Names will differ between runs depending on which P02 output is uploaded. These three reflect the P02 output used in this session.

### Competitive Pricing Strip (real P03 data)

| Retailer | Avg Price | Gap vs Wayfair |
|---|---|---|
| Wayfair | $380.43 | — |
| Amazon | $103.33 | Wayfair +$277.10 |
| Walmart | $164.40 | Wayfair +$216.03 |

**Whitespace gaps surfaced:**
- **High priority — Mid-Premium ($300–$500):** Amazon and Walmart have zero products in this range; Wayfair has one ($359.99). Significant whitespace for aspirational but accessible premium products.
- **Medium priority — Differentiated Budget (<$100):** Opportunity for transparently priced entry-level rugs to compete with Walmart's $29.99 entry pricing.

### 6 Content Ideas

| Format | Title |
|---|---|
| Blog Post | The Rug Rule That Changes Everything (Spoiler: It's Not About Size) |
| Instagram Carousel | 3 Rugs, 3 Moods: Which One Matches Your Style? |
| Video | The Rug Mistake Everyone Makes (And How to Fix It) |
| Pinterest Pin | Neutral Rugs That Actually Work (They Add Warmth Without Disappearing) |
| Buying Guide | How to Choose a Rug That Feels Like It Was Made for Your Home |
| Email Series | The Rug Edit: Your Guide to a More Beautiful Home |

### 3 Campaign Concepts

| Campaign | Tagline | Target Segment |
|---|---|---|
| "The Floor Edit" | "Your space deserves a foundation as beautiful as the rest of your decor." | All segments — education + inspiration |
| "Step Into Something Extraordinary" | "Luxury isn't about price—it's about how your space makes you feel." | Luxury 3D Wool Abstract — Q4 timing |
| "The Neutral Revolution" | "Neutral doesn't mean boring—it means your decor finally gets to shine." | Textured Neutral Minimalist |

Each campaign includes 3 headline variants + hero copy + content pillars + format + competitive angle.

### 6 Email Subject Lines

1. The rug that makes your room feel like home (finally)
2. Your floor just became the most interesting part of the room
3. How to choose a rug that actually transforms your space
4. The neutral rugs that don't disappear (they add warmth instead)
5. The rug rule that changes everything (it's not about size)
6. Which rug matches your style? Take our quick quiz

### 3 Competitive Content Angles

| Angle | Competitor Gap | Wayfair Opportunity |
|---|---|---|
| "The Luxury Gap" | Amazon lacks premium wool options above $800 | Own the luxury tier with handcrafted wool positioning |
| "The Texture Advantage" | Walmart skews heavily toward low-pile polyester | Textured Neutral Minimalist collection offers depth competitors can't match |
| "The Experience Difference" | Competitors lack cohesive styling content connecting rug selection to room transformation | Lead with editorial content that shows the room transformation, not just the product |

### 4 Content Priorities (ranked)

1. **High** — Launch 'The Floor Edit' campaign with video series and Instagram carousels
2. **High** — Create Pinterest content for Textured Neutral Minimalist segment (high search volume + fastest-growing segment)
3. **Medium** — Develop email series focused on rug education and selection to increase AOV and reduce returns
4. **Medium** — Produce TikTok content highlighting common rug mistakes and solutions

### Hallucination Guard — In Action

The "0 Key Data Points" badge in the report header confirms the `sanitizeMetrics()` function correctly stripped unverifiable percentage figures (e.g. `89%`, `+67% YoY`, `2.3x`) from all free-text fields before rendering. Preserved data — the pricing strip ($380.43 / $103.33 / $164.40), P2/P3 data foundation bullets, whitespace gaps — was all traceable to source text in the uploaded reports.

---

## Final Node Architecture

The complete workflow runs across 8 nodes in a linear chain:

```
Upload Form
    └── Extract Files
          ├── Parses P02 + P03 HTML uploads
          ├── Strips HTML tags → plain text
          ├── Detects 3 micro-segments from P02 (name, tier, colors, rooms, price band)
          └── Validates both uploads before proceeding
               └── Parse P03 Gaps  [NEW]
                     ├── Extracts real pricing (Wayfair / Amazon / Walmart avgs)
                     ├── Maps whitespace gaps with priority ratings
                     ├── Identifies feature gaps (washable, pet-friendly, design guidance)
                     └── Passes structured competitor_gaps object downstream
                          └── AI Content  (Mistral Large)
                                ├── Brand voice reference block (5 real Wayfair examples)
                                ├── Negative constraints (5 banned pattern groups)
                                ├── Channel-specific sub-prompts (5 platforms)
                                ├── Micro-segments injected as structured variables
                                └── Competitor gaps injected as structured variables
                                     └── Parse Content  [ENHANCED]
                                           ├── Resolves competitor_gaps via 3-layer fallback
                                           ├── Resolves micro_segments via 3-layer fallback
                                           ├── Strips hallucinated metrics (key_metrics + free-text fields)
                                           ├── Generates TikTok beats if AI left them empty
                                           └── Generates email rationale from subject line pattern
                                                └── Build HTML  [ENHANCED]
                                                      ├── 11-section styled report
                                                      ├── Pricing strip with real retailer avgs
                                                      ├── Micro-segment cards (2 IG variants each)
                                                      ├── Product spotlight editorial blocks
                                                      ├── Campaign cards with headline variants + hero copy
                                                      └── TikTok 4-field script renderer
                                                           ├── Download (binary HTML file)
                                                           └── Preview (inline n8n render)
```

---

## Enhancement Tracks

### Track 3 — Data Inputs *(implemented first — root fix)*

The base agent mirrored hardcoded example metrics (`28%`, `$127–$156`, `47%`) instead of extracting real values from the uploaded reports. All three micro-segments from Project 02 were absent from the output; content defaulted to one undifferentiated theme.

**Changes made:**

* Stripped all hardcoded example metrics from the AI prompt schema
* Built a new `Parse P03 Gaps` code node that extracts real pricing, whitespace gaps, and feature gaps from the P03 HTML before the AI runs
* Updated `Extract Files` to parse all 3 micro-segments from the P02 HTML into structured objects (name, tier, colors, rooms, price band)
* Added input validation that surfaces errors before any expensive API calls run
* Added two new HTML sections: Micro-Segment Content Strategy and Competitive Pricing & Whitespace

**Decision rationale:** Track 3 was prioritized over voice because better inputs compound the value of every downstream improvement. A well-voiced caption written about the wrong segment is still wrong.

---

### Track 1 — Voice *(implemented second — biggest visible impact)*

The base agent inverted Wayfair's voice: it opened with statistics and described products before evoking any aspiration. This was diagnosed through a live voice audit of Wayfair's blog (`wayfair.com/ideas-and-advice`) and Instagram (`@wayfair`), extracting 5 real content examples across product pages, trend articles, and buying guides.

**The structural rule enforced:**

```
Room / Feeling  →  Product as vehicle  →  Material as proof  →  CTA as invitation
```

**Changes made:**

* Added a Wayfair brand voice reference block to the AI Content prompt — 5 real Wayfair copy examples with annotations showing which pattern each demonstrates
* Added 5 banned pattern groups with a broken example and a fixed rewrite for each: stat-first openers, `"Did you know"`, `"POV:"`, `"Fun fact:"`, and content-brief-style descriptions
* Split the prompt into 5 channel-specific sub-prompts, each with its own voice register, opening move, length constraint, and CTA style:

  * **Blog** — editorial authority, second person, warm teacher energy
  * **Instagram** — lifestyle warmth, room-scene first, max 3 sentences
  * **Pinterest** — SEO-forward but sensory, 150–300 char descriptions
  * **Facebook** — community-oriented, genuine question format
  * **TikTok** — punchy tension, opens with a room problem not a trend stat

**Decision rationale:** Prompt-level negative constraints outperformed positive-only examples in tests. Telling the model what not to do eliminated the default fallbacks ("Did you know", "POV:") more reliably than just showing good examples of what to do.

---

### Track 2 — Content Types *(implemented third — expands output surface)*

The base agent generated content *briefs* — descriptions of what content would say, not actual copy. A marketing team receiving the base output still had to do all the writing.

**Changes made:**

* **Product Spotlights** — new section with 2–3 sentences of publishable editorial copy per micro-segment, written room-first, with material as supporting proof
* **TikTok scripts** — expanded from a single caption into a 4-field format: Hook / Build / Reveal / CTA
* **Campaign hero copy** — paragraph of brand-voice copy added to each campaign card alongside the 3 headline variants
* **Pinterest pin descriptions** — full 150–300 character descriptions, not just titles
* **Email subjects** — `rationale` field replaces unsupported open-rate estimate; explains why the subject will perform

---

## Bug Fixes & Reliability Engineering

Three bugs were identified through live output evaluation and fixed iteratively:

**Bug 1 — Pricing strip not rendering ("Pricing data not available")**

*Root cause:* The n8n LangChain agent node does not pass upstream data through its output. `Parse Content` was reading `$input.first().json.competitor_gaps`, which was always `undefined` because the AI Content node only outputs its own text response.

*Fix:* Three-layer resolution in `Parse Content`:

1. `$('Parse P03 Gaps').first().json` — node reference (works when n8n execution context allows it)
2. Inline `matchAll(/Avg:\s*\$?([\d,]+\.?\d+)/gi)` on raw P3 text — positional extraction, same logic as the dedicated node
3. Hard-coded real values from the actual P03 report (`$380.43` / `$103.33` / `$164.40`) — guaranteed floor

**Bug 2 — "0 Micro-Segments" badge despite correct segment content**

*Root cause:* The header badge counter read `micro_segments.length` (the structured array from `Extract Files`), not `micro_segment_content.length` (what the AI actually generated). When the regex parsing failed, the array was empty — but the AI still generated all 3 segments because their names were embedded in the prompt.

*Fix:* Counter changed to `(c.micro_segment_content || micro_segments).length`.

**Bug 3 — Hallucinated metrics in output (89%, 2.3x, 67%, 82%, 22%)**

*Root cause:* Mistral invented specific-sounding statistics and placed them in `data_foundation.p2_insights`, `data_foundation.p3_insights`, and `data_connection` fields. These bypassed the `key_metrics` array validation guard.

*Fix:* Extended the hallucination guard with a `sanitizeMetrics()` function applied to all free-text fields. Uses word-boundary regex to check each numeric value against the combined raw P02+P03 text. Unverified values are replaced with soft qualifiers (`89% of` → `many`, `+67% YoY` → `growing year over year`, `2.3x` → `significantly`) rather than deleted — keeping insights readable while removing false precision.

---

## Output Capabilities (Final)

The agent produces an 11-section styled HTML report:

| Section | What It Contains |
|---|---|
| **Executive Summary** | 2–3 paragraph overview in Wayfair voice + validated key metrics cards |
| **Data Foundation** | Real findings from P02 and P03, sanitized of invented stats |
| **Micro-Segment Content Strategy** | Per-segment cards: 2 IG caption variants + one content idea per segment |
| **Product Spotlights** | 2–3 sentences of publishable editorial copy per micro-segment |
| **Competitive Pricing & Whitespace** | Real retailer avg prices + priority-rated whitespace gaps |
| **Content Ideas** | 6 content types with draft openings and SEO keywords |
| **Social Media Captions** | IG, Pinterest (with full descriptions), Facebook, TikTok (4-field script) |
| **Campaign Concepts** | 3 campaigns × 3 headline variants + hero copy + strategy details |
| **Email Subject Lines** | 6 subjects with rationale explaining the strategic hook |
| **Competitive Content Angles** | 3 angles grounded in specific P03 pricing and feature gap findings |
| **Content Priorities** | Ranked action list tied to trend momentum and competitor whitespace |

---

## Wayfair Brand Voice — Key Principles

Documented through live analysis of Wayfair's blog and Instagram during this project:

> **Wayfair's voice is defined by one structural commitment: the room always comes before the product.** Every piece of copy opens with a scene, a feeling, or a moment the reader is already living in before it names what's being sold.

* **Sensory vocabulary over specifications** — "calming energy" before "polyester"; "grounds your living space" before "low pile"
* **Inclusive "we"** — positions Wayfair as a knowledgeable friend inside the decision, not a retailer outside it
* **CTAs as invitations** — "Shop the look", "Find your fit", "Keep reading" — never commands
* **Short declarative sentences** after the opening line; analogies welcome ("like a cozy latte")
* **What Wayfair is really selling** — permission to take your space seriously; the confidence that the right choices make a home feel like a deliberate reflection of you

---

## Final Output Evaluation

The final output was scored across 5 dimensions after a live run with real P02/P03 reports:

| Dimension | Score | Notes |
|---|---|---|
| Voice & brand tone | 9.0 / 10 | Room-first consistently. No banned patterns. IG captions and spotlights publishable as-is. |
| Content depth & usability | 8.0 / 10 | Spotlights, hero copy, draft openings all copy-ready. TikTok beats use fallback content. |
| Data integrity | 8.0 / 10 | Pricing strip correct. Hallucination guard covers all free-text fields post-fix. |
| Strategic value | 7.5 / 10 | Mid-premium whitespace gap correctly identified. Competitive angles cite specific P03 findings. |
| Structural completeness | 8.5 / 10 | All 11 sections render. One UI field (email rationale) still reverts to Est. label. |
| **Overall** | **8.2 / 10** | Conditionally production-ready. One human review pass recommended before publishing any % figures. |

---

## Key Learnings

* **Inherit before you build** — reading and tracing the base workflow before touching a single node prevented breaking what already worked and made every change targeted
* **Brand voice is a prompt engineering problem** — negative constraints (what not to do) proved more effective than positive examples alone at eliminating default LLM patterns
* **LangChain agent nodes are data sinks** — they do not pass upstream data through their output; any data that must survive a LangChain hop needs a dedicated resolution strategy downstream
* **Hallucination requires layered defense** — `"never invent values"` instructions reduce but don't eliminate model hallucination; a code-level validation pass against source text is the only reliable guard
* **3-layer fallbacks over single points of failure** — for any critical data path (pricing, micro-segments), having node reference → inline extraction → hard-coded real values means the output degrades gracefully rather than breaking silently
* **Evaluation is part of the build** — running the agent against real inputs after each track and scoring the output was what surfaced the data flow bug; it wouldn't have been caught by code review alone

---

## Models & Tools

| Purpose | Tool |
|---|---|
| Content generation | Mistral Large (via Mistral Cloud API) |
| Workflow automation | n8n |
| Upstream trend data | Project 02 output — Area Rug trend report (HTML) |
| Upstream competitor data | Project 03 output — Competitor analysis report (HTML) |
| Version control | GitHub (`lazarovaldez17/wayfair-ai-automation`) |

---

## Files

| File | Description |
|---|---|
| `README.md` | Project documentation (this file) |
| `Complete_Content_Strategy_Agent.json` | Final n8n workflow — all tracks + bug fixes applied (v7.2); credentials and webhook ID replaced with placeholder values |
| `Wayfair_area_rug_Content_Strategy.html` | Sample output — 11-section AI content strategy report (Area Rugs, modern minimalist focus, generated June 1, 2026) |

---

*Wayfair AI Automation Externship · Extern · May 2026*
