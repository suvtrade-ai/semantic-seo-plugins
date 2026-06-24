---
name: monthly-audit
description: Monthly SEO audit and content refresh pipeline for a Semantic SEO local business site. Pulls real ranking data from Google Search Console via Composio, checks indexation status, identifies thin or stale content, finds new keyword opportunities, refreshes competitor gap analysis, and produces a monthly report. Use when the user says "monthly audit", "/monthly-audit", "monthly check", "check my rankings", "update my content", "refresh my site", "monthly SEO report", "content audit", "how is my site performing", or when a scheduled task triggers this skill.
tools:
  - Read
  - Write
  - WebSearch
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

| Page | GSC Position | GSC Impressions | Content Status | Action |
|------|-------------|----------------|---------------|--------|

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

---

### Updated monthly report structure (add these sections):

After the existing 8 sections, append:

```
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
