---
name: semantic-research
description: Runs Phase 2 of the Semantic SEO framework — Semantic Research using Koray Tuğberk Gübür's methodology. Identifies and validates entities, builds attribute matrices (popular, prominent, relevant), maps entity relationships, and defines context vectors and topical boundaries. Produces 02-semantic-research.md. Use when the user says "semantic research", "/semantic-research", "entity research", "attribute research", "entity validation", "entity map", "context vectors", "topical boundary", "what entities should my site cover", or "start Phase 2".
tools:
  - Write
  - WebSearch
---

# Semantic Research (Phase 2)

If `01-foundation.md` exists, read it first to understand the business context. Otherwise ask for: business type, services, city.

**If the site already has a GSC property connected via Composio:** before starting entity research, call `GOOGLE_SEARCH_CONSOLE_SEARCH_ANALYTICS_QUERY` with dimensions=["query"] for the past 3 months to pull real query data. Use these queries to:
- Identify which entities Google already associates this site with (queries containing entity names = validated by real user behavior)
- Find missing entity coverage gaps (competitor queries this site does not rank for — these are Phase 2 entity research targets)
- Detect any semantically out-of-scope queries the site ranks for (potential topical boundary violations — flag them)

Then perform the following and write to `02-semantic-research.md`:

## Entity Identification

**Minimum entity checklist — verify ALL of these categories are covered before proceeding:**

```
[ ] Business entity (the shop itself — named exactly as it appears on GBP and legal documents)
[ ] Service entities (one distinct entity per core service — do NOT merge "suit tailoring" and "shirt tailoring")
[ ] Location entities (city, specific neighborhoods, hotel districts, tourist areas served)
[ ] Process entities (the steps involved — measurement, fitting, alteration, delivery)
[ ] Material/fabric entities (specific fabric types offered — wool, linen, silk, etc.)
[ ] Problem entities (the customer's starting problem — ill-fitting suit, event deadline, damaged garment)
[ ] Credential/trust entities (years in business, certifications, awards — with specific values)
[ ] Person entities (owner name, master tailor name — if publicly nameable)
```

If ANY category has zero entries, research it before moving forward. An empty category means that topic area will be missing from the topical map.

List all entities in a table: `Entity | Type | Relevance (Core / Supporting / Edge) | Source`

## Entity Validation

For each entity, validate via three methods — record all three results:

**Source-based validation:**
- Does Wikipedia have an article on this entity?
- Does schema.org have a defined type for it?
- Does Google show a Knowledge Panel when you search the entity name alone?

**SERP-based validation:**
- Do at least 3 of the top 10 competitor pages mention this entity by name?
- Do PAA questions reference this entity?
- Do related searches include this entity name?

**Contextual validation:**
- If you removed this entity from the site's content, would the site feel incomplete to a customer?

Mark each entity: `Validated (passes 2+/3 methods) | Partially Validated (passes 1/3) | Excluded (reason)`

Only Validated and Partially Validated entities advance to the Attribute Matrix.

## Attribute Matrix

### Step 1: Data collection

Search the primary query for each core entity (e.g. "custom tailor Bangkok"). Open the **top 20 ranking pages**. For each page, record every distinct attribute mentioned — these are factual properties of the entity: turnaround time, price range, fabric types, process steps, location details, certifications, languages spoken, and so on.

Create a raw count table before scoring:

```
| Attribute              | Pages mentioning it (out of 20) | Appears in H1/first paragraph of top 3? | Appears in PAA questions? |
|------------------------|--------------------------------|------------------------------------------|---------------------------|
| [attribute name]       | [count]                        | Yes / No                                 | Yes / No                  |
```

### Step 2: Scoring rules — apply these exact thresholds

| Score | Definition | Threshold | Action |
|-------|-----------|-----------|--------|
| **Popular** | Most pages in the niche cover this — Google expects to see it | Present on **8 or more of the top 20** pages | Must appear on your site or your pages will look incomplete |
| **Prominent** | Top-ranking pages treat this as central — it anchors their content | Appears in the **H1 or first 200 words** of **3 or more of the top 5** pages | Place in your H1 or opening paragraph — omitting it signals to Google you're off-topic |
| **Relevant** | Underused but semantically connected — differentiation opportunity | Appears in **PAA questions** but on **fewer than 4 of the top 20** ranking pages | Cover this and you fill a gap competitors haven't claimed |

A single attribute can score multiple levels — e.g. "turnaround time" may be both Popular (on 14/20 pages) AND Prominent (in the H1 of 4 of top 5).

### Step 3: Output table

```
| Entity          | Popular Attributes (8+/20 pages) | Prominent Attributes (in H1/first 200w of top 3) | Relevant Attributes (PAA but <4/20 pages) | Your Coverage Gap |
|-----------------|----------------------------------|---------------------------------------------------|-------------------------------------------|-------------------|
| [entity name]   | [list]                           | [list]                                            | [list]                                    | [missing attrs]   |
```

"Your Coverage Gap" = Popular or Prominent attributes that are currently absent from your content.

## Entity Relationship Map

**Filter rule: only map a relationship if it changes what content to put on a page or where to put an internal link.** Skip relationships that are obvious or don't affect site structure.

A relationship is worth mapping if at least one of these is true:
- It determines which page owns a topic (e.g. "express tailoring" has-a "turnaround time" → turnaround data lives on the express page, not the main tailoring page)
- It creates a parent-child linking requirement (e.g. "bespoke suit" is-type-of "tailoring service" → suit page links up to main tailoring service page)
- It prevents content duplication (e.g. "hotel fitting" is-a "service" → hotel fitting gets its own page, not a section on the main services page)
- It determines schema nesting (e.g. "Master Tailor [Name]" is-a "Person" → use Person schema on the About page, not on product pages)

**Relationship types to use:**
- `is-a` — subset/classification
- `has-a` — attribute ownership
- `is-type-of` — parent category
- `located-in` — geography
- `used-for` — application
- `performed-by` — person/role
- `associated-with` — semantic proximity but no hierarchy

Output table:

```
| Entity A              | Relationship | Entity B              | Content implication                          |
|-----------------------|--------------|-----------------------|----------------------------------------------|
| Hotel fitting service | is-a         | Service               | Needs own page, links to main service page   |
| Turnaround time       | has-a        | Express tailoring     | Turnaround data lives on express service page |
| [Master Tailor name]  | performed-by | Tailoring service     | Person schema on About page, bio links to services |
```

## Context Vectors & Topical Boundary

List topics as In-scope vs Out-of-scope:

**In-scope:** topics the site MUST cover to be topically authoritative for this niche (derived from the entity list above)

**Out-of-scope:** topics that would dilute authority — adjacent services this business doesn't offer, location areas outside the service area, garment types not made

Then write 5–10 context signal sentences. Each sentence must follow S-P-O structure (Subject → Predicate → Object), name entities explicitly, and contain one fact per sentence:

Examples of strong context signal sentences:
- "[Business name] provides bespoke tailoring services in [City]."
- "[Business name] offers turnaround times of [X] to [Y] days for custom suits."
- "[Business name] uses [fabric types] sourced from [origin] for men's suiting."

These sentences become the factual spine that all page content must be consistent with.

Produce `02-semantic-research.md` and confirm it was saved.


---

## Integration: seo-cluster (Phase 2 → Phase 3 bridge)

After completing the entity and attribute matrices, pass the core entity list to `seo-cluster` to pre-validate your topical clusters against real Google SERP overlap before committing to the Phase 3 topical map:

```
/seo cluster plan [primary entity keyword]
```

Why here: SERP overlap analysis reveals which entity combinations Google treats as the same topic (7+ shared results = same page) vs. distinct topics (0-1 shared results = separate pages). This prevents building a topical map where you accidentally plan two pages that should be one.

**Output:** `cluster-plan.json` and `cluster-plan.md` — feed these into Phase 3 (`topical-map`) as the starting architecture. The topical map skill will refine it using Koray's hub-and-spoke methodology.

---

## Integration: seo-dataforseo (optional, if API connected)

If DataForSEO is connected, use it during entity validation to pull real keyword volume and competition data for the core entities:

```
/seo dataforseo keywords [entity name] [city]
```

This replaces the manual SERP volume estimation step with real data. Use the returned CPC and competition scores to prioritize which entity attributes to cover first (highest volume, lowest competition = fastest wins).

**If DataForSEO is NOT connected:** proceed with WebSearch-based SERP estimation. Flag this in `02-semantic-research.md` as "estimated volumes — upgrade with DataForSEO when available."

---

## CoR Step — EAV Triple Validation + Knowledge-Based Trust

After completing the Attribute Matrix, run an EAV validation pass on all discovered entities. Reference `cor/frameworks/eav-architecture.md` and `cor/audits/knowledge-graph-validation.md`.

### EAV Triple Validation Checklist

For every key fact in `02-semantic-research.md`, verify:

```
[ ] Explicit naming — entity is named directly, no ambiguous pronouns
[ ] Specific values — numbers and measurements, not "many" or "some"
[ ] Definitive modality — "is" for facts, "can" for possibilities, "should" for advice
[ ] Consistent units — same measurement system throughout
[ ] Source-backed — fact can be verified from Wikipedia, schema.org, or authoritative source
```

### Knowledge-Based Trust (KBT) Consistency Check

All EAV triples extracted here become the **locked source of truth** for the entire site. Add a KBT table to `02-semantic-research.md`:

```
| Entity | Attribute | Locked Value | Source |
|--------|-----------|-------------|--------|
| [CE]   | Price range | [exact range] | [source] |
| [CE]   | Location | [exact address] | NAP |
| [CE]   | Turnaround time | [X days] | owner confirmed |
```

**KBT killers to flag and reject:**
- Vague values: "affordable", "quick", "high quality" → replace with specific numbers
- Contradictory values: if Page A says "2–3 days" and Page B says "3–5 days" → standardize now
- Missing citations: claims without a verifiable source → add source or remove

**Why this step matters:** Google's Knowledge-Based Trust patent scores sites on consistency of facts across pages. Contradictions increase Cost of Retrieval and reduce trust scores. See `cor/concepts/cost-of-retrieval.md` for the full breakdown.

Append the KBT table to `02-semantic-research.md` before Phase 3.
