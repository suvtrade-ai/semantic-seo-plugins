---
name: monthly-audit
description: Monthly SEO audit and content refresh pipeline for a Semantic SEO local business site. Pulls real ranking data from Google Search Console via Composio, checks indexation status, identifies thin or stale content, finds new keyword opportunities, refreshes competitor gap analysis, and produces a monthly report. Use when the user says "monthly audit", "/monthly-audit", "monthly check", "check my rankings", "update my content", "refresh my site", "monthly SEO report", "content audit", "how is my site performing", or when a scheduled task triggers this skill.
tools:
  - Read
  - Write
  - WebSearch
  - Bash
---

# Monthly SEO Audit & Content Refresh

Run this once per month. Produces `monthly-report-[YYYY-MM].md`.

## Before starting

Read these files if they exist:
- `05-technical-local.md` — site structure and full page list
- `04-content-system.md` — content inventory
- `weekly-log.md` — what was published last month
- Previous month's report — to compare progress

Ask the user: "What is your site's domain?" (e.g. mysuitbangkok.co or garmanisuit.com) — needed for all GSC calls.

---

## Section 0 — SerpAPI + Tavily Pre-Audit Research

Run before pulling GSC data. These two steps give you the external view (what Google sees) to compare against the internal view (what GSC reports).

### 0a — SerpAPI: check live rankings for key pages

```python
import os, requests, json

SERPAPI_KEY = os.environ.get("SERPAPI_API_KEY")

# Read tracked keywords from 03-topical-map.md or weekly-log.md
# Define the pillar keywords to check rankings for
tracked_keywords = [
    "[service] [city]",           # primary money keyword
    "bespoke [service] [city]",   # secondary
    "[service] price [city]",     # commercial intent
    # add more from the topical map
]

rankings = {}
for kw in tracked_keywords:
    params = {
        "engine": "google",
        "q": kw,
        "api_key": SERPAPI_KEY,
        "num": 20,
        "gl": "th",       # set to business country
        "hl": "en"
    }
    resp = requests.get("https://serpapi.com/search", params=params).json()
    organic = resp.get("organic_results", [])

    # Find the site's position
    domain = "[business-domain.com]"  # replace with actual domain
    position = next(
        (r["position"] for r in organic if domain in r.get("link", "")),
        None
    )
    local_pack = resp.get("local_results", {}).get("places", [])
    in_local_pack = any(domain in p.get("website", "") for p in local_pack)

    rankings[kw] = {
        "position": position,
        "in_local_pack": in_local_pack,
        "top3_urls": [r.get("link") for r in organic[:3]],
        "paa": [q["question"] for q in resp.get("related_questions", [])],
    }
    status = f"Position {position}" if position else "Not in top 20"
    pack = "✓ In local pack" if in_local_pack else "✗ Not in local pack"
    print(f"  {kw}: {status} | {pack}")

with open("rank-check.json", "w") as f:
    json.dump(rankings, f, indent=2)
print("Saved rank-check.json")
```

Include `rank-check.json` data in the monthly report — compare position to last month's report.

### 0b — Tavily: content freshness research

For each page that's been live for 3+ months and hasn't been updated, run Tavily to check if the topic has new data:

```python
from tavily import TavilyClient
import os

tavily = TavilyClient(api_key=os.environ.get("TAVILY_API_KEY"))

# Pages to check for freshness (read from 04-content-system.md)
stale_pages = [
    {"slug": "/bespoke-suits-bangkok/", "topic": "bespoke suit tailoring Bangkok 2025"},
    # add more from content inventory
]

freshness_data = {}
for page in stale_pages:
    result = tavily.search(
        page["topic"],
        max_results=5,
        search_depth="advanced",
        include_answer=True,
        days=90  # look for content published in the last 90 days
    )
    freshness_data[page["slug"]] = {
        "recent_content_found": len(result.get("results", [])) > 0,
        "summary": result.get("answer", ""),
        "sources": [r["url"] for r in result.get("results", [])],
    }
    flag = "⚠ NEW DATA AVAILABLE" if freshness_data[page["slug"]]["recent_content_found"] else "✓ Still current"
    print(f"  {page['slug']}: {flag}")

print("\nPages needing content refresh based on Tavily freshness check:")
for slug, data in freshness_data.items():
    if data["recent_content_found"]:
        print(f"  → {slug}: {data['summary'][:100]}")
```

Pages where Tavily finds recent competing content = **priority refresh queue** for this month.

---

## Section 1 — Real ranking data from Google Search Console

Use Composio's Google Search Console tools to pull real data. Do NOT use WebSearch to guess rankings.

**Step 1a — Identify the GSC property**

Call `GOOGLE_SEARCH_CONSOLE_LIST_SITES` via Composio. Find the property matching the user's domain. Note the exact `site_url` string (e.g. `https://mysuitbangkok.co/` or `sc-domain:garmanisuit.com`) — it must be used exactly in all subsequent calls.

**Step 1b — Query-level ranking data**

Call `GOOGLE_SEARCH_CONSOLE_SEARCH_ANALYTICS_QUERY` with:
- `site_url`: exact property string from Step 1a
- `start_date`: first day of last month (YYYY-MM-DD)
- `end_date`: last day of last month (YYYY-MM-DD)
- `dimensions`: ["query"]
- `row_limit`: 100
- `search_type`: "web"

From the response, extract:
- Top 20 queries by impressions
- Top 20 queries by clicks
- Queries ranked position 1–3 (primary wins)
- Queries ranked position 4–10 (low-hanging fruit — close to page 1)
- Queries ranked position 11–20 (page 2 — needs content work)
- Queries with high impressions but low CTR (< 3% CTR at position < 10 = title/meta issue)

**Step 1c — Page-level performance data**

Call `GOOGLE_SEARCH_CONSOLE_SEARCH_ANALYTICS_QUERY` with:
- Same date range
- `dimensions`: ["page"]
- `row_limit`: 100

From the response, identify:
- Top performing pages (most clicks)
- Pages with zero or near-zero clicks despite impressions (content or intent mismatch)
- Pages not appearing at all (possible indexation issue)

**Step 1d — Indexation check for key pages**

For each P1 page from `04-content-system.md`, call `GOOGLE_SEARCH_CONSOLE_INSPECT_URL` with the full URL. Check:
- Is the page indexed?
- Any coverage issues (noindex, blocked by robots.txt, redirect issues)?
- Last crawl date (if > 30 days, Google is not crawling it regularly — internal links may be weak)

**Step 1e — Sitemap health**

Call `GOOGLE_SEARCH_CONSOLE_LIST_SITEMAPS` with the site_url. Check:
- Submitted count vs. indexed count (large gap = crawl issue)
- Any sitemap errors or warnings
- When the sitemap was last downloaded by Google

**Build the ranking table:**

| Page | URL | Impressions | Clicks | Avg Position | CTR | Status | vs Last Month |
|------|-----|------------|--------|-------------|-----|--------|--------------|
| Homepage | / | [real data] | [real data] | [real data] | [real data] | Indexed | ↑/↓/= |
| [Service 1] | /[slug]/ | ... | ... | ... | ... | Indexed | |
| [Service 2] | /[slug]/ | ... | ... | ... | ... | Not indexed | ⚠️ |

Tell the user what the numbers mean in plain English: "Your homepage is ranking at position 4.2 for [query] — you're close to the top 3. Here's what to do..."

---

## Section 2 — CTR opportunity list

From the GSC query data, identify queries where:
- Position is 1–10 (you're on page 1)
- CTR is below expected benchmark:
  - Position 1: expect > 25% CTR
  - Position 2–3: expect > 10% CTR
  - Position 4–7: expect > 5% CTR
  - Position 8–10: expect > 2% CTR

For each under-performing query, the fix is almost always the title tag or meta description. Produce specific rewrites:

| Query | Page | Position | Actual CTR | Expected CTR | Current Title | Suggested Title | Suggested Meta |
|-------|------|----------|-----------|-------------|--------------|----------------|----------------|

---

## Section 3 — Content quality audit

Review each published page. Flag as Thin or Stale using the same criteria as before, but now cross-reference against the GSC page-level data:

- A page with zero impressions after 8+ weeks = either not indexed or no topical demand — check with INSPECT_URL first
- A page with impressions but position > 20 = content is relevant but weak — needs expansion and attribute coverage improvement
- A page with good impressions and position 4–10 = close to winning — targeted improvement here gives fastest ROI

**Attribute coverage check (run for every page at position 4–20):**

Open `02-semantic-research.md`. For the page's primary entity, check the attribute matrix:
1. Are ALL Popular attributes (present on 8+ of top 20 competitor pages) covered on this page? Missing popular attributes are the most likely cause of underperformance.
2. Are ALL Prominent attributes (in H1 or first 200 words of top 3 competitor pages) either in the H1 or the opening paragraph? If not, the page signals weak topical relevance to Google.
3. Is at least 1 Relevant attribute (PAA but <4 of top 20 pages) covered? If not, the page has no differentiation from competitors.

Add a column to the content audit table: `Missing Attributes (from 02-semantic-research.md)` — list by name. These become the specific content additions for that page's next refresh.

| Page | GSC Position | GSC Impressions | Content Status | Missing Attributes | Action |
|------|-------------|----------------|---------------|-------------------|--------|

---

## Section 4 — Low-hanging keyword wins

From the GSC query data, find all queries where:
- The site ranks position 11–20 (page 2)
- The query has 50+ impressions in the past month

These are pages that need content improvement, not new pages. For each:
- Identify which page is ranking for this query
- Check: does the page's H1 match this query's intent?
- Check: does the page cover the prominent attributes for this query?
- Write specific improvements to make

Also use WebSearch to find new queries not yet in the site's GSC data — topics competitors rank for that the site doesn't touch yet.

---

## Section 5 — Competitor gap refresh

Re-check the 2–3 main competitors. Search their domain on Google and note new content published this month. Also pull the GSC data for queries where the site has impressions but position > 10 — these are queries where competitors are outranking. Search those queries and analyze what the top-ranking competitor page has that this site's page is missing (entity coverage, attribute depth, content length, schema).

---

## Section 6 — Internal link audit

Check for orphan pages: pages with zero or one inbound internal link. Cross-reference the GSC page-level data — pages with very low crawl frequency (last crawled > 30 days) often have weak internal linking.

Check for broken internal links by verifying that destination slugs still exist.

---

## Section 7 — GBP check via Zernio

**If Zernio MCP is connected** (tools prefixed with `zernio__` will be available):

**Step 7a — Verify GBP account is connected:**
Call `zernio__accounts_list` and look for `platform: "googlebusiness"`. If found, note the `accountId`.

**Step 7b — Check GBP posts published this month:**
Call `zernio__posts_list` with `status: "published"` and filter results for `platform: "googlebusiness"`. Cross-check count against `weekly-log.md` — flag any weeks where a GBP post is missing.

**Step 7c — Pull and respond to new reviews:**
Call `zernio__list_inbox_reviews` to fetch reviews. For each review:
- Note: reviewer name, star rating, text snippet, date, whether a reply exists
- For any unanswered review: draft a response (professional, personalized — reference something specific from the review text, thank them, invite return visit)
- Post each response via `zernio__reply_to_inbox_post` with the review's post ID

**Step 7d — Check for unanswered Q&A:**
Zernio does not manage GBP Q&A directly. Tell the user: "Check your GBP Q&A section manually — go to your Google Business Profile → Q&A → answer any new questions. This takes 2 minutes."

**If Zernio is not connected or no GBP account found:**
- Cross-reference `weekly-log.md` — were all GBP posts published?
- Tell the user: "Go to your GBP dashboard → Reviews — respond to any new reviews within 24 hours. Check Q&A for new questions."
- Ask: "How many new reviews this month? Average rating?"

**Report GBP summary:**
| Metric | This Month |
|--------|-----------|
| New reviews | [from zernio__list_inbox_reviews or user answer] |
| Reviews responded to | [count auto-responded via Zernio] |
| GBP posts published | [from posts_list or weekly-log.md] |
| Missing post weeks | [flag any] |

---

## Section 8 — Monthly report

Write `monthly-report-[YYYY-MM].md` with:

**Executive Summary** — 4–5 plain-English sentences: total clicks vs last month, top ranking wins, biggest opportunity, one thing that needs fixing urgently.

**GSC Rankings Table** (from Section 1 — real data)

**CTR Opportunity List** (from Section 2 — specific title/meta rewrites)

**Content Published This Month** (from `weekly-log.md`)

**Indexation Issues** (from Section 1d — any pages not indexed, fix instructions)

**Sitemap Health** (from Section 1e — submitted vs indexed gap)

**Content Actions for Next Month** — prioritized by GSC impact (work on position 4–10 pages first)

**New Keyword Opportunities** (from Section 4)

**GBP Summary** (from Section 7)

**Goals for Next Month:**
1. [specific, measurable — e.g., "Improve [query] from position 6.2 to top 3 by expanding [page] with [specific attributes]"]
2. [specific — e.g., "Fix indexation on [page] by checking robots.txt and adding 2 internal links to it"]
3. [specific — e.g., "Get [X] new Google reviews"]

---

After producing the report, tell the user the top 3 actions in plain language and which will have the fastest ranking impact.

---

## Step 6 — Index health audit

After reviewing rankings and content, run the indexing health check:

1. Invoke `/indexing` in monthly audit mode
2. The indexing skill will:
   - Re-inspect all P1 and P2 pages via URL Inspection API
   - Compare current verdicts against `indexing-log.md`
   - Flag any pages that are FAIL or NEUTRAL after 7+ days
   - Alert on any pages that have dropped from PASS to NEUTRAL/FAIL (deindexing signal)
   - Re-submit flagged pages to the Indexing API
3. Append the index health report to the monthly audit output

**Key things to check:**
- All P1 pages must show PASS — if any show NEUTRAL or FAIL, treat as priority
- Check `lastCrawlTime` — P1 pages should be crawled at least every 14 days on an active site
- If a previously-indexed page has dropped: check Rank Math settings site-wide for accidental noindex, check if a plugin update changed robots headers

**If GSC is connected (native):**
Use URL Inspection API to get real-time indexation data for all tracked pages.

**If GSC is not connected:**
Check manually in GSC → Coverage report and flag any pages showing "Excluded" or "Error" status.

---

## Step 7 — Google review management

Reviews are a direct local ranking signal. Check and respond every month minimum.

### Check new reviews

**If Zernio MCP is connected:**
```
zernio__reviews_list(platform: "googlebusiness", since: "[last audit date]")
```

**If not connected:** Ask user — "How many new Google reviews did you receive this month? Any negative ones I should help respond to?"

### Draft responses for unanswered reviews

For each unanswered review:

**Positive review response formula (3 sentences max):**
1. Thank them by name + reference the specific service or garment they mention
2. One natural brand/entity mention ("at The Outfit" or "our Bangtao workshop")
3. Invite them back or reference a related service

**Negative review response formula:**
1. Acknowledge without being defensive
2. Take it offline: "Please contact us directly at [phone/WhatsApp] so we can make this right"
3. Never argue, never over-explain

**Review velocity check:**
- Target: respond to all reviews within 48 hours
- Flag if: fewer than 2 new reviews this month (for an active business, this is below normal)
- If review count is low: suggest review request templates for WhatsApp follow-up after each completed order

### Monthly review summary

Add to the monthly report:
```
## Reviews — [month]
- New reviews: X (Google: X)
- Average rating: X.X
- Responded: X / X
- Notable themes in positive reviews: [e.g. fast turnaround, quality fabric]
- Notable themes in complaints: [e.g. pricing, wait time]
- Action: [any follow-up needed]
```

---

## Step 8 — Schema health check

Once per month, spot-check schema on P1 pages — plugin updates can silently break structured data.

1. Run Rich Results Test on homepage and top 2 P1 service pages
2. Check GSC → Enhancements report for any new schema errors flagged by Google
3. If errors appear: go to Rank Math → Schema on that page and fix missing fields
4. Add result to monthly report: "Schema: OK / X errors found on Y pages"

**floorLevel check (multi-story buildings):**
If the business is in a multi-story building, verify `floorLevel` is still present in the homepage LocalBusiness JSON-LD every month. WordPress plugin updates (Rank Math, theme updates) can silently overwrite custom schema fields.

Check: view page source → search for `floorLevel` → confirm it appears OUTSIDE the `PostalAddress` block with the correct floor value.

If missing after an update: re-add via Rank Math → Schema → Custom Schema → paste the `"floorLevel": "[value]"` property back into the JSON-LD block. Flag in monthly report: "floorLevel schema: OK / Missing — re-added"

---

## Integration: claude-blog + claude-seo (monthly enhancement tools)

### Section 9 — Content refresh with blog-rewrite

For any page flagged as Thin or Stale in Section 3, or any page ranked position 4-10 in need of a targeted push:

```
/blog rewrite [page file or URL]
```

`blog-rewrite` is the full-rewrite tool: replaces fabricated statistics with sourced data, applies answer-first formatting, adds missing FAQ schema, injects citation capsules for AI visibility, adds information gain markers, and updates freshness signals (published/modified dates in schema).

**Priority order for rewrites:**
1. Pages at position 4-10 with 50+ impressions (fastest ranking impact)
2. Pages with zero clicks after 8+ weeks (indexation or intent mismatch)
3. Pages with CTR below benchmark for their position (title/meta issue — fix those first, rewrite only if CTR fix alone doesn't work within 2 weeks)

After rewriting, run `/blog geo` and `/blog factcheck` on the updated file, then re-publish and re-submit to Google Indexing API.

### Section 10 — Per-page deep audit with seo-page

For top 3 traffic pages, run a deep single-page audit monthly:

```
/seo page [page URL]
```

Covers on-page elements, content quality, technical meta tags, schema, images, and performance in one pass. Produces a prioritized fix list. Use this to catch issues that the GSC data doesn't surface directly (broken canonical, wrong schema type, duplicate meta, image CLS).

### Section 11 — AI citation audit with blog-geo

For the top 2 content pages and the homepage:

```
/blog geo [page URL]
```

Monthly AI visibility check: are these pages getting cited in ChatGPT, Perplexity, Google AI Overviews? The score tracks over time — a dropping score signals that competitors have published stronger citation capsules. Add or strengthen capsules if the score drops below 70.

### Section 12 — Backlink health (Month 2 onward)

From Month 2 after launch:

```
/seo backlinks [domain]
```

Pull referring domains, anchor text distribution, new links acquired this month, and any toxic links. Flag if:
- No new referring domains in the past 30 days (outreach needed)
- Anchor text over-optimized (>30% exact match on any target keyword)
- Toxic links appearing (disavow candidates)

Add backlink summary to the monthly report. Connect to local link building opportunities identified in `05-technical-local.md`.

### Section 13 — DataForSEO keyword opportunities (if connected)

If DataForSEO API is connected:

```
/seo dataforseo keywords [domain] --new-opportunities
```

Pulls keyword gaps not yet in GSC data — queries competitors rank for that this site doesn't. Supplements the manual Section 4 low-hanging keyword wins with real volume data. Prioritize new content opportunities by: search volume × (1 / keyword difficulty).

**If DataForSEO is not connected:** skip this section. The GSC Section 4 data is sufficient for the first 6 months.

### Section 14 — Content quality batch scoring with blog-analyze

Run once per month across ALL published blog posts and supporting articles to identify which content needs a rewrite:

```
/blog analyze --batch --sort score
```

`blog-analyze` scores every article on a 100-point system across 5 categories:
- **Content Quality (30pts):** depth, specificity, answer-first format, expert signals
- **SEO (25pts):** title/meta tag quality, heading structure, internal links, keyword density
- **E-E-A-T (15pts):** author signals, entity clarity, sourced claims
- **Technical (15pts):** schema, canonical, OG tags, mobile formatting
- **AI Citation Readiness (15pts):** citability score, citation capsules, Q&A structure

Also runs AI content detection: burstiness (sentence length variance), 17 AI phrase patterns (em dashes, "delve into", "it's worth noting", etc.), and type-token ratio — flags articles at risk of Google quality filters.

**Action thresholds:**

| Score | Action |
|-------|--------|
| 85–100 | OK — monitor only |
| 70–84 | Targeted fix (usually schema or intro paragraph) |
| < 70 | Schedule `/blog rewrite` this month (add to Section 9 queue) |

**Cross-reference with GSC data from Section 1c:** a low-scoring article with high impressions is the highest-priority rewrite — it's getting traffic but underperforming because the content is weak.

Sorting ascending (`--sort score`) shows the weakest articles first. Fix or rewrite in that order.

**Add to monthly report:**

```
## Blog Quality Scores — [YYYY-MM]
| Article | Score | Weakest Category | AI Detection | Action |
|---------|-------|-----------------|-------------|--------|
| [article title] | [N]/100 | [e.g. E-E-A-T] | Clean / Flagged | OK / Fix / Rewrite |
```

---

### Updated monthly report structure (add these sections):

After the existing 8 sections, append:

```
## Blog Quality Scores — [YYYY-MM]
| Article | Score | Weakest Category | AI Detection | Action |
|---------|-------|-----------------|-------------|--------|
| [article title] | [N]/100 | [category] | Clean / Flagged | OK / Fix / Rewrite |

## Content Refresh Log — [month]
- Pages rewritten: [list]
- Blog-rewrite improvements: [before/after position if known]

## AI Citation Scores — [month]
- [Page 1]: [score] (up/down from last month)
- [Page 2]: [score]

## Backlink Summary — [month] (Month 2+)
- New referring domains: X
- Total referring domains: X
- Top new links: [list]
- Issues: [any toxic or over-optimized anchor text]
```

---

## CoR Step — 100-Point Semantic Compliance Audit

Run this audit on the **top 3 traffic pages** each month (or any page flagged with declining rankings). Reference `cor/audits/semantic-compliance.md` for full scoring sheets.

### Scoring System (6 categories, 100 points total)

**1. Contextual Flow — 20 points**

| Criterion | Points |
|-----------|--------|
| H1→H2→H3 logical order, attributes flow Unique→Root→Rare | 4 |
| Contextual bridges between sections (no abrupt topic jumps) | 4 |
| Subordinate text: first sentence after every heading directly answers heading | 4 |
| Discourse integration: anchor segments connect paragraphs | 4 |
| Correct attribute priority ordering throughout | 4 |

**2. EAV Quality — 20 points**

| Criterion | Points |
|-----------|--------|
| All key facts in explicit S-P-O triple structure | 4 |
| All values specific — no "many", "some", "often", "affordable" | 4 |
| Values consistent with 01-foundation.md KBT table | 4 |
| Correct modality: "is" for facts, "can" for possibilities | 4 |
| Claims have verifiable sources or owner confirmation | 4 |

**3. Information Density — 15 points**

| Criterion | Points |
|-----------|--------|
| One unique fact per sentence | 3 |
| No filler words (actually, basically, really, very, overall…) | 3 |
| No redundant restatements | 3 |
| Average sentence length < 30 words | 3 |
| No fluff paragraphs (every paragraph adds at least one EAV triple) | 3 |

**4. Link Compliance — 15 points**

| Criterion | Points |
|-----------|--------|
| Same anchor text used max 3× per page | 3 |
| All internal links placed AFTER entity/concept is defined | 3 |
| Annotation text (surrounding text) semantically supports each link | 3 |
| Total internal links per page < 150 | 3 |
| Links in main content area outweigh boilerplate links | 3 |

**5. Format Compliance — 15 points**

| Criterion | Points |
|-----------|--------|
| Content format matches search intent (list for how-to, table for comparison) | 3 |
| Lists preceded by intro sentence stating item count | 3 |
| Featured Snippet answer present: < 40 words directly after H2 | 3 |
| Visual hierarchy: most important content prominent | 3 |
| Semantic HTML used (article, section, h1-h3, ul/ol, table) | 3 |

**6. Technical Compliance — 15 points**

| Criterion | Points |
|-----------|--------|
| Schema correctly implemented, no validation errors | 3 |
| Image alt text varies with topical attributes (not repeated H1) | 3 |
| URL structure: lowercase, hyphens, entity name, max 5 words | 3 |
| Page TTFB < 100ms (check GSC crawl stats) | 3 |
| DOM node count < 1,500 | 3 |

### Compliance Score Thresholds

| Score | Status | Action |
|-------|--------|--------|
| 90–100 | Excellent | Monitor monthly |
| 85–89 | Good — meets threshold | Minor improvements only |
| 70–84 | Needs improvement | Schedule rewrite within 30 days |
| < 70 | Failing | Priority rewrite this week |

Add the score for each audited page to `monthly-report.md` as: `Page | Score | Category Breakdown | Priority Action`
