---
name: foundation
description: Runs Phase 1 of the Semantic SEO framework — Foundation & Strategy. Performs market research, market segmentation, ICP construction, search intent mapping, and competitor semantic gap analysis for a local business. Produces a 01-foundation.md deliverable. Use when the user says "foundation", "/foundation", "market research", "ICP research", "search intent map", "competitor gap analysis", "who is my target customer", or "start Phase 1".
tools:
  - Write
  - WebSearch
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
| Query contains "book", "appointment", "contact", "call", "WhatsApp" | Transactional | Contact page or service page with click-to-call / WhatsApp button |
| Query contains "best", "top", "vs", "review", "recommend", "quality" | Commercial | Service page with trust signals: reviews, photos, credentials, comparison |
| Query contains "how to", "what is", "what fabric", "how long does", "can a tailor" | Informational | Supporting blog article (do NOT target with a service page) |
| Query contains business name | Navigational | Homepage or About page |
| Google shows a **map pack** for this query | Local intent dominant | GBP + service page with strong NAP consistency |
| Google shows a **featured snippet** for this query | Informational | Blog post structured to match the snippet format (list / paragraph / table) |

Build the intent table across the 15–20 most important queries for this business:

```
| Query                            | Intent         | SERP has map pack? | Target page type          | CTA on that page        |
|----------------------------------|----------------|--------------------|---------------------------|-------------------------|
| custom tailor [city]             | Transactional  | Yes                | Service page + GBP        | Call / WhatsApp / Book  |
| how long does tailoring take     | Informational  | No                 | Blog article              | Internal link → service |
| [city] suit fitting              | Commercial     | Yes                | Service page              | Portfolio + Reviews     |
```

## Competitor Gap Analysis

For each of the 3 competitors extracted in Market Research, fill this framework:

```
Competitor: [Name]

Semantic gaps (topics they mention but cover shallowly):
- [H2 heading they have but with thin content — e.g. "Pricing" section with no actual prices]
- [Process section missing steps]

Topical gaps (entire topic areas absent from their site):
- [e.g. No dedicated express tailoring page]
- [e.g. No fabric guide]
- [e.g. No hotel/in-room fitting page]

Entity gaps (entities they fail to name explicitly):
- [e.g. No mention of specific fabric brands]
- [e.g. No mention of specific neighborhoods or hotel names they serve]
- [e.g. No certification names]

Opportunity rating: High / Medium / Low
(High = gap is also a high-search-intent query; Low = gap exists but nobody is searching for it)
```

After completing all 3 competitors, consolidate into a **Priority Gap Table**:

```
| Gap topic                    | Found in N/3 competitors | Search intent | Priority |
|------------------------------|--------------------------|---------------|----------|
| [topic]                      | [0/1/2/3 competitors]    | [type]        | High/Med/Low |
```

Gaps absent from all 3 competitors with Transactional or Commercial intent = **highest priority content to build first**.

---

## CoR Step — Central Entity Attribute Mapping

After the competitor gap analysis, define the **EAV attribute hierarchy** for the Central Entity (CE). This feeds directly into Phase 2 entity research and every content brief. Reference `cor/frameworks/eav-architecture.md` for full rules.

**Attribute types (priority order — highest to lowest):**

| Type | Definition | Rule |
|------|-----------|------|
| **Unique Attributes** | Features that uniquely identify this CE — no other entity shares the same attribute+value | Place early in all content. These prove expertise. |
| **Root Attributes** | Essential for basic definition of the CE — without these, the entity is incomplete | Must be covered on every core page. |
| **Rare Attributes** | Specific details not commonly found — proves depth of knowledge | Use to differentiate from competitors. |

**Output a table in `01-foundation.md`:**

```
Central Entity (CE): [business type + city]

| Attribute Type | Attribute | Value | Notes |
|----------------|-----------|-------|-------|
| Unique | [e.g. speciality not offered by competitors] | [specific value] | |
| Unique | | | |
| Root | [e.g. price range, turnaround time, location] | [specific value] | |
| Root | | | |
| Root | | | |
| Rare | [e.g. specific technique, certification, niche process] | [specific value] | |
| Rare | | | |
```

**KBT rule:** once these values are written here, they are locked. Every page, schema markup, and GBP listing must use the exact same values. Contradictions across pages destroy Knowledge-Based Trust (KBT) and increase Cost of Retrieval. See `cor/concepts/cost-of-retrieval.md`.

Produce the file `01-foundation.md` and confirm it was saved.
