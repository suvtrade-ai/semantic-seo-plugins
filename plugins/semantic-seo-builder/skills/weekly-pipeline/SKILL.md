---
name: weekly-pipeline
description: Weekly content operations pipeline for a Semantic SEO local business site. Each run produces one fully written, SEO-optimized blog post or supporting article (with semantic brief, full content, internal links, and schema), plus a matching Google Business Profile post. Tracks what has been published. Use when the user says "weekly pipeline", "/weekly-pipeline", "write this week's blog post", "weekly content", "publish a blog post", "GBP post", "what do I publish this week", "weekly SEO task", or when a scheduled task triggers this skill.
tools:
  - Read
  - Write
  - WebSearch
  - Bash
---

# Weekly Content Pipeline

This skill runs one complete content publishing cycle. Each run = one article + one GBP post.

## Before writing

1. Read `03-topical-map.md` to find the next unpublished supporting article in the content queue. Start from the highest-priority pillar and work down.
2. Read `04-content-system.md` to check which pages are already published (look for "Published" status in the content inventory table).
3. Read `02-semantic-research.md` for entity and attribute context.
4. If a `weekly-log.md` file exists, read it to find which articles have already been published this week/month and pick the next one.

If none of these files exist, ask: "What is your site about, and what topic should this week's article cover?"

---

## Step 1 — Pick this week's topic

Select the next unpublished Tier 3 (supporting content) article from the topical map. Prioritize:
- Articles that support P1 service pages (they pass the most authority)
- Topics with questions in Google's "People Also Ask" for the niche
- Topics that competitors have thin or missing coverage on

Tell the user: "This week I'll write about: [topic]. This supports your [pillar page] and helps you rank for [related query]."

---

## Step 1b — Tavily pre-research (always run before writing)

Before building the brief, use Tavily to gather current, specific data about this topic. This prevents generic content and surfaces statistics, recent developments, and PAA angles that competitors haven't covered.

```python
from tavily import TavilyClient
import os, json

tavily = TavilyClient(api_key=os.environ.get("TAVILY_API_KEY"))

topic = "[this week's article topic]"  # e.g. "how long does bespoke suit tailoring take Bangkok"
business_context = "[service] [city]"  # from 01-foundation.md

# Deep research on the specific topic
topic_research = tavily.search(
    f"{topic}",
    max_results=8,
    search_depth="advanced",
    include_answer=True,
    include_raw_content=False
)

# Find what competitors are publishing recently
competitor_angle = tavily.search(
    f"{business_context} {topic.split()[0]} blog site:*.com -site:wikipedia.org",
    max_results=5,
    search_depth="basic",
    include_answer=False
)

print("=== Topic Research Summary ===")
print(topic_research.get("answer", "No synthesis — use individual results"))
print("\n=== Competitor Angles ===")
for r in competitor_angle.get("results", []):
    print(f"  - {r['title']}: {r['url']}")
    print(f"    {r.get('content', '')[:150]}")

# Save for use in Step 2
with open("weekly-research.json", "w") as f:
    json.dump({
        "topic": topic,
        "tavily_summary": topic_research.get("answer", ""),
        "tavily_sources": [r["url"] for r in topic_research.get("results", [])],
        "competitor_urls": [r["url"] for r in competitor_angle.get("results", [])],
    }, f, indent=2)
print("Saved weekly-research.json")
```

**Use the Tavily output to:**
- Identify 1–2 statistics or data points that can be cited in the article
- Find competitor articles that rank for this topic — note their angle and what they're missing
- Pull the Tavily answer synthesis as the seed for the article's opening answer-first paragraph
- Identify if this topic has recent developments (last 3–6 months) that should be mentioned

Read `weekly-research.json` in Step 2 when building the content brief.

---

## Step 2 — Write the semantic content brief

Before writing, build a brief (silent — don't show the user unless they ask):

- Primary entity: [entity]
- Supporting entities: [list from entity map]
- Attributes to cover: pull from the attribute matrix in `02-semantic-research.md`
  - **Popular attributes** (present on 8+ of top 20 pages) → must be covered or the article looks incomplete
  - **Prominent attributes** (in H1/first 200 words of 3+ of top 5 pages) → include in H1 or opening paragraph
  - **Relevant attributes** (PAA questions but <4/20 pages) → include at least one — this is what differentiates the article from generic competitor content
- Target intent: Informational (supporting content is always informational)
- Query path: [what query brings users here] → [what they search next] → [how this article bridges that]
- Word count: 800–1,500 words (supporting articles)
- Internal links out: 2 links — one to the parent subtopic page, one to a sibling article or service page
- Schema: Article

---

## Step 3 — Write the article

Write the full article following these rules:

**Structure:**
- H1: matches the primary query for this topic (informational intent)
- First 100 words: directly answer the question the H1 poses (answer-first format)
- H2 sections: cover popular attributes, then prominent attributes, then relevant/differentiating attributes
- H2 "FAQ": 3–5 People Also Ask questions answered in 40–60 words each
- Closing paragraph: transition sentence that links to the relevant service page ("If you're looking for [service] in [city], [link to service page]...")

**Semantic requirements:**
- Mention the business entity naturally (not forced) at least 2 times
- Cover all prominent attributes from the attribute matrix
- Include at least 1 relevant attribute that most competitor pages miss
- All internal links use entity-based anchor text (not "click here")

**What NOT to include:**
- Filler content ("In today's world..." / "In this article, we will explore...")
- Repetition of the H1 in the first sentence
- Generic advice that has nothing to do with this specific niche/city

---

## Step 4 — Format for WordPress

Provide the article in clean markdown with:
- The H1 as the title
- All H2/H3 properly nested
- Internal link placeholders: [LINK TO: /slug/] — "anchor text here"
- Image placeholder: [IMAGE: describe what image to use here]
- Schema note at the bottom: "Add Article schema via Rank Math on this post"

Tell the user: "Copy this content into WordPress → Posts → Add New. Replace [LINK TO: /slug/] with actual links to those pages. Add an image where indicated. Set the URL slug to: /[suggest the slug]/"

---

## Step 4b — Generate hero image (requires GEMINI_API_KEY)

After writing the article, generate a geotagged hero image via the `/image-gen` skill:

1. Extract the primary entity from the article title (e.g., "bespoke linen suit Phuket")
2. Invoke `/image-gen` with:
   - Type: `blog-hero`
   - Entity: the primary article topic
   - Location: business location from `01-foundation.md`
3. The image-gen skill will:
   - Generate via Kie Nano Banana API
   - Convert to WebP at quality 72 (target under 100KB)
   - Save as `images/{article-slug}-hero.webp`
   - Return an `<img>` tag with SEO alt text
4. Provide the `<img>` tag alongside the article content for the user to insert as the featured image in WordPress.

If no Kie API key is available, leave the `[IMAGE: ...]` placeholder and note: "Run `/image-gen` with your Kie API key to generate a hero image for this article."

---

## Step 4c — Submit to Google Indexing API + crawl check (optional, requires service account)

After publishing to WordPress, if `gsc-service-account.json` exists in the project root, submit the live URL for priority indexing:

1. Get the live URL of the published post from WordPress (returned by the publish step or confirmed by the user)
2. Invoke `/indexing` in weekly mode:
   - Submit the URL to the Google Indexing API
   - Log the submission in `indexing-log.md`
3. After 30–60 minutes (or at the start of the next session), run the crawl check:
   - Call URL Inspection API on the submitted URL
   - Report the verdict (PASS / NEUTRAL / FAIL)
   - If FAIL: diagnose (check noindex tag, robots.txt, Rank Math settings)
   - Append result to `indexing-log.md`

If no service account is configured, tell the user:
"To submit this page to Google automatically, run `/indexing` and follow the 5-minute service account setup. Until then, you can manually request indexing in GSC → URL Inspection → Request Indexing."

---

## Step 5 — Write and publish the GBP post

Write a matching Google Business Profile post (300 words max):

- Tone: friendly, direct, local — not corporate
- Opening: addresses a common customer problem or question (related to this week's article)
- Middle: 2–3 sentences of value (what they should know)
- Closing: call to action with phone number or link
- Post type: "What's new" or "Offer" if there's a promotion

**If Zernio MCP is connected** (tools prefixed with `zernio__` will be available):

First confirm the GBP account is connected:
- Call `zernio__accounts_list` — look for an account with `platform: "googlebusiness"` and note its `accountId`

Then publish the GBP post directly:
```
zernio__posts_create(
  content: "[the 300-word GBP post text]",
  platform: "googlebusiness",
  publish_now: true
)
```

Log the returned `post_id` in `weekly-log.md`. Tell the user: "Your Google Business post has been published automatically."

**If Zernio is not connected or no GBP account is found:**
Tell the user: "Post this to your Google Business Profile: Google Business → Posts → Add update → What's new → paste this text → Add a photo → Publish"

---

## Step 6 — Update the weekly log

Append to `weekly-log.md`:

```
## Week of [YYYY-MM-DD]

- Article: [title]
- URL slug: /[slug]/
- Word count: [N]
- Images: [N] generated — [list filenames]
- GBP post: [published / pending]
- Indexing: [submitted / manual]
- AI Citation score: [N]/100
- Drift check: [pass / issues found]
```

---

## Integration: claude-blog + claude-seo (weekly production pipeline)

The weekly pipeline produces one complete article per run. Here is the full enhanced workflow using claude-blog and claude-seo skills:

### Step W1 — Write with blog-write

```
/blog write [this week's topic from content-calendar.md]
```

Produces a full article with answer-first formatting, sourced statistics, FAQ schema, internal link zones, and image placeholders. Uses the persona defined in `PERSONA.json` and the brand voice from `BRAND.md` (if present) for consistent voice.

### Step W2 — Fact-check with blog-factcheck

```
/blog factcheck [new article file]
```

Verifies every statistic. Fix any claims scoring 0.0 before proceeding. Target: all claims at 0.7+ confidence with live source URLs.

### Step W3 — SEO validation with blog-seo-check

```
/blog seo-check [new article file]
```

Pass/fail checklist: title tag length and keyword placement, meta description, heading hierarchy, internal and external link audit, canonical URL, Open Graph tags, Twitter Card, URL structure, image alt text. Fix all failures before publishing.

### Step W4 — Schema with blog-schema

```
/blog schema [new article file]
```

Generates BlogPosting/Article JSON-LD, FAQPage schema from the FAQ section, BreadcrumbList. Paste into Rank Math → Custom Schema on the published page.

### Step W5 — GEO check with blog-geo

```
/blog geo [new article file]
```

AI Citation Readiness score. If below 70, add or strengthen 2-3 citation capsules — self-contained answer paragraphs that AI systems can lift directly. Aim for at least one capsule per major H2 section.

### Step W6 — Images with image-gen

```
/image-gen [new article file]
```

Generates section-appropriate images via Kie API: hero + inline illustrations. Converts to WebP. Place images and update the article's image tags before publishing.

### Step W7 — GBP post (existing step)

After the article is published, write and post the matching GBP update linking to the article URL. This is the existing weekly GBP post step — no change.

### Step W8 — Repurpose with blog-repurpose (optional, monthly rotation)

```
/blog repurpose [published article URL]
```

On the first week of each month, repurpose the previous month's highest-traffic article into: a Twitter/X thread, a LinkedIn post, and a YouTube script outline. This extends reach without extra research time.

### Step W9 — SEO drift check with seo-drift

```
/seo drift [domain] --check
```

After publishing, run a quick drift check to confirm the new article didn't accidentally change any baseline SEO elements on existing pages (title tags, canonical URLs, noindex settings). WordPress plugin conflicts or theme updates during publishing can silently break existing pages.

**If no baseline exists yet:** run `/seo drift [domain] --baseline` once to capture the first snapshot. Subsequent weekly runs compare against it.

---

### Step W5b — Algorithmic Authorship pass (CoR writing quality gate)

After `/blog write` produces the article draft and **before** factcheck, run the Algorithmic Authorship pass. Reference `cor/frameworks/algorithmic-auth