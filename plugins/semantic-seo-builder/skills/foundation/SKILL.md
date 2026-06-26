---
name: foundation
description: Runs Phase 1 of the Semantic SEO framework — Foundation & Strategy. Performs market research, market segmentation, ICP construction, search intent mapping, and competitor semantic gap analysis for a local business. Produces a 01-foundation.md deliverable. Use when the user says "foundation", "/foundation", "market research", "ICP research", "search intent map", "competitor gap analysis", "who is my target customer", or "start Phase 1".
tools:
  - Write
  - WebSearch
  - Bash
---

# Foundation & Strategy (Phase 1)

Ask the user for:
1. Business name and niche/service type
2. City and service area
3. Full address including building name if applicable
4. **Is the business inside a multi-story building?** If yes, what floor? (e.g. "2nd Floor", "5th Floor") — needed for `floorLevel` schema property
5. Names of 2–3 main competitors
6. Does a website already exist for this business?

Store the floor level answer. If the business is in a multi-story building (mall, office tower, shared commercial building), the `floorLevel` property MUST be added to the LocalBusiness schema in Phase 5. This is a Google-introduced schema.org property that disambiguates businesses sharing the same building — critical for businesses inside malls, plazas, or multi-tenant towers.

## Step 0a — Existing site baseline (existing site rebuilds only)

**If the user confirmed an existing website in the intake:**

Invoke the `seo-plan` skill first:
- Run `/seo plan [domain]` to get a quick competitive baseline: current rankings, content gaps, what's working, industry-specific opportunities
- This takes 5–10 minutes and produces a snapshot that feeds directly into the Koray research below
- Do NOT skip this for rebuilds — it prevents duplicating what already works and reveals what's failing

Then proceed with GSC data pull below.

**If new site (no existing domain):** skip this step entirely. Go to Step 0b.

---

## Step 0b — Customer voice research (always run)

Before any market research, run `blog-discourse` to capture what real customers say about this business type in this city:

Invoke `/blog discourse [business type] [city]` with `--days 30`

For example: `/blog discourse custom tailor Bangkok --days 30`

This searches Reddit, TripAdvisor reviews, Google reviews, Facebook groups, TikTok comments, Instagram hashtags, YouTube videos, and travel forums. It produces `DISCOURSE.md` containing:
- What customers complain about (pain points → ICP problems)
- What they praise (trust signals → content angles)
- The exact language they use (vocabulary → H1s, meta descriptions, ad copy)
- Influencer and blogger mentions (link building targets)
- Trending questions (FAQ content, PAA opportunities)

**After `DISCOURSE.md` is produced:** read it before writing the ICP profiles and search intent map. Use the real customer language — not generic marketing copy.

---

## Step 0c — Brand voice (always run)

Invoke `/blog persona` to define the brand voice before any content is written:
- Tone dimensions: Funny↔Serious, Formal↔Casual, Respectful↔Irreverent, Enthusiastic↔Matter-of-fact
- Sets vocabulary tier, sentence length, contraction frequency
- Saves as a persona file used automatically by blog-write, blog-brief, and weekly-pipeline

Ask the user: "How would you describe the brand voice? (e.g. professional and precise, friendly and approachable, luxury and exclusive)" — use their answer to configure the persona.

---

## Step 0d — SerpAPI + Tavily + Apify Research Layer (always run)

This step runs three APIs in parallel to collect real SERP data, market intelligence, and competitor content before any manual analysis. It replaces guesswork with structured evidence.

**Required env vars:** `SERPAPI_API_KEY`, `TAVILY_API_KEY`, `APIFY_API_TOKEN`

### 1. SerpAPI — SERP signals for primary keyword

```python
import os, requests, json

SERPAPI_KEY = os.environ.get("SERPAPI_API_KEY")
primary_query = f"{service} {city}"  # e.g. "custom tailor Bangkok"

params = {
    "engine": "google",
    "q": primary_query,
    "api_key": SERPAPI_KEY,
    "num": 20,
    "gl": "th",        # set to business country ISO code
    "hl": "en"
}
data = requests.get("https://serpapi.com/search", params=params).json()

organic     = data.get("organic_results", [])
paa         = [q["question"] for q in data.get("related_questions", [])]
local_pack  = data.get("local_results", {}).get("places", [])
related     = [s["query"] for s in data.get("related_searches", [])]
has_map     = len(local_pack) > 0
has_snippet = any("snippet" in r for r in organic[:3])

print(f"SERP signals for: {primary_query}")
print(f"  Local pack present: {has_map} ({len(local_pack)} businesses)")
print(f"  Featured snippet: {has_snippet}")
print(f"  PAA questions ({len(paa)}): {paa}")
print(f"  Related searches: {related}")
print(f"  Top organic URLs:")
for r in organic[:5]:
    print(f"    {r.get('position')}. {r.get('title')} — {r.get('link')}")
```

**Use the output to:**
- Set `has_map_pack` flag (drives GBP priority in Phase 5)
- Pre-populate the PAA → FAQ section of relevant pages
- Add related searches as secondary keyword targets in the intent map
- Use top 5 organic URLs as competitor extraction targets below

### 2. Tavily — Market intelligence synthesis

```python
from tavily import TavilyClient
import os

tavily = TavilyClient(api_key=os.environ.get("TAVILY_API_KEY"))

# Market overview
market = tavily.search(
    f"{service} {city} market overview pricing competition 2025",
    max_results=8,
    search_depth="advanced",
    include_answer=True
)

# Customer language
customers = tavily.search(
    f"who uses {service} in {city} tourist expat local review",
    max_results=5,
    search_depth="advanced",
    include_answer=True
)

print("=== Market Intelligence ===")
print(market.get("answer", "No synthesis available"))
print("\n=== Customer Profiles ===")
print(customers.get("answer", "No synthesis available"))
```

**Use the output to:**
- Seed the Market Segmentation section with real demographic data
- Validate or challenge the DISCOURSE.md customer segments
- Identify pricing data points that competitors publish (or hide)

### 3. Apify google-search-scraper — Bulk competitor content extraction

```python
from apify_client import ApifyClient
import os

apify = ApifyClient(os.environ.get("APIFY_API_TOKEN"))

run = apify.actor("apify/google-search-scraper").call(run_input={
    "queries": primary_query,
    "resultsPerPage": 10,
    "maxPagesPerQuery": 1,
    "countryCode": "TH",        # set to business country
    "languageCode": "en",
    "includeUnfilteredResults": False,
})

serp_items = list(apify.dataset(run["defaultDatasetId"]).iterate_items())
for item in serp_items:
    for result in item.get("organicResults", [])[:5]:
        print(f"  {result.get('position')}. {result.get('title')}")
        print(f"     URL: {result.get('url')}")
        print(f"     Desc: {result.get('description', '')[:120]}")
```

**Use the output to:**
- Confirm top 5 competitor URLs (cross-reference with SerpAPI)
- Extract meta descriptions to map competitor positioning angles
- Identify which pages Google is rewarding (page type: homepage / service page / directory listing)

### Store research results

After all three APIs complete, write a `research-raw.json` file:

```python
import json
with open("research-raw.json", "w") as f:
    json.dump({
        "primary_query": primary_query,
        "has_map_pack": has_map,
        "local_pack_businesses": [p.get("title") for p in local_pack],
        "paa_questions": paa,
        "related_searches": related,
        "top5_organic_urls": [r.get("link") for r in organic[:5]],
        "tavily_market_summary": market.get("answer", ""),
        "tavily_customer_summary": customers.get("answer", ""),
    }, f, indent=2)
print("Saved research-raw.json")
```

Read `research-raw.json` when filling in Market Research, Search Intent Map, and ICP Profiles below. Do not re-run the APIs — use the cached data.

---

**If a website already exists:** use Composio's `GOOGLE_SEARCH_CONSOLE_LIST_SITES` to find the connected GSC property. Then call `GOOGLE_SEARCH_CONSOLE_SEARCH_ANALYTICS_QUERY` with dimensions=["query"] for the past 3 months. Use the real query data to:
- Identify which queries the site already ranks for (skip researching these in competitor gap — focus on what's MISSING)
- Identify queries where the site has impressions but low clicks (intent mismatch — flag for content brief improvement)
- Identify queries where position > 20 (site is visible but weak — these become topical map priorities)

Then perform the following and write everything to `01-foundation.md`:

## Market Research

Search the primary service + city query (e.g. "custom tailor Bangkok"). Open the **top 3 ranking pages** and extract the following template for each:

```
Competitor Extraction — [Competitor Name] ([URL])
| Field                        | Value |
|------------------------------|-------|
| H1 (exact text)              |       |
| Page type                    | service page / homepage / listing |
| Estimated word count         | (count visible body text sections) |
| Schema types present         | LocalBusiness / FAQPage / Service / Review / etc. |
| FAQ count                    | (number of FAQ items on page) |
| Price transparency           | Yes — shows prices / Yes — shows range / No |
| Express / turnaround mentioned | Yes / No |
| Hotel or mobile service mentioned | Yes / No |
| Trust signals                | (certifications, years in business, awards, photos) |
| Topics this page covers      | (list H2 headings or main topics) |
| Topics this page is MISSING  | (what a customer would want to know that's absent) |
```

After extracting all 3 competitors, read across the "Topics missing" column. Any gap that appears in 2+ competitors is a **priority content opportunity** — cover it first.

Also note SERP-level signals:
- Is there a **local pack / map pack** at the top? → GBP + location pages are critical
- Is there a **featured snippet**? → note its exact format (paragraph / list / table) — structure content to match that format
- Are there **People Also Ask** boxes? → extract all questions, add to FAQ section of the relevant page

## Market Segmentation

**Derive segments from DISCOURSE.md first.** Do not invent segments from assumptions.

Process:
1. Read `DISCOURSE.md`. Group complaints into themes (e.g. "prices not listed", "long wait", "poor communication")
2. Each complaint cluster = a distinct pain point → name the customer type experiencing it
3. Group praise themes (e.g. "fast turnaround", "perfect fit", "speaks English well") → these become trust signal priorities
4. Pull the exact vocabulary customers use — these words go into H1s and meta descriptions verbatim

Then create a segment table covering 4–5 distinct types:

```
| Segment Name         | Who They Are                              | Trigger (what makes them search now) | Primary Fear / Objection         | Value to Business |
|----------------------|-------------------------------------------|--------------------------------------|----------------------------------|-------------------|
| [name]               | [travel status, occasion, demographics]   | [specific event or situation]        | [what stops them from booking]   | High / Med / Low  |
```

Each segment must differ in at least two of: trigger, fear, query language, or page they land on first. If two rows look the same, merge them.

## ICP Profiles

Write 2–3 ICP profiles using this exact template. Reference DISCOURSE.md for language and trigger events.

```
### ICP: [Name — e.g. "The Event-Driven Expat"]

- **Who**: [demographics, travel status, occasion — be specific]
- **Awareness stage**: Problem-aware / Solution-aware / Product-aware
  (Problem-aware = knows they have a clothing need but hasn't searched for tailors yet)
  (Solution-aware = searching for tailoring options)
  (Product-aware = comparing specific tailors)
- **Trigger event**: [the specific event that made them search RIGHT NOW — e.g. "wedding in 3 weeks", "just arrived for 6-month work assignment"]
- **Search query sequence**:
  1. [their first search — often broad]
  2. [their refinement search after first results]
  3. [their conversion search — the one just before they contact a tailor]
- **First landing page they reach**: [homepage / service page / blog post]
- **Primary objection**: [the one thing that stops them converting — e.g. "I can't see prices", "I don't know if they speak English", "I don't know if one fitting is enough"]
- **Conversion signal**: [what makes them pick up the phone or book — e.g. "sees a price range", "reads a review mentioning their nationality", "sees photos of suits like theirs"]
- **Content they need to see**: [the specific page or section that resolves their objection]
```

## Search Intent Map

**Classification rubric — apply these rules for every query before assigning intent:**

| Rule | If you see this… | Intent → Target with |
|------|-----------------|----------------------|
| Query contains "price", "cost", "how much", "rates", "quote" | Transactional | Service page with visible price range or at minimum "from X" |
| Query contains "near me", "in [city]", "in [neighborhood]", "[city] tailor" | Transactional / Local | GBP listing + service page with address + LocalBusiness schema |
| Query co