# Phase 4 — Content System

## Goal
Turn the topical map into a production-ready content operation. Every page is written against a semantic brief — not a keyword brief. Semantic briefs tell writers WHAT the page needs to cover (entities, attributes, intent, query path), not just what keyword to rank for.

## Content Inventory Table

List every planned page from the processed topical map as a row:

| # | Page Title | URL Slug | Page Type | Tier | Primary Entity | Target Intent | Word Count | Priority |
|---|-----------|----------|-----------|------|---------------|---------------|------------|----------|
| 1 | Homepage | / | Homepage | — | Business entity | Navigational + Transactional | 800–1200 | P1 |
| 2 | [Service Name] | /[service-slug]/ | Service | Pillar | Service entity | Transactional | 1500–2000 | P1 |
| 3 | [Subtopic Name] | /[pillar]/[subtopic]/ | Subtopic | Tier 2 | Subtopic entity | Commercial | 1000–1500 | P2 |
| ... | | | | | | | | |

Priority levels:
- **P1**: Revenue-generating pages — build these first (homepage, service pages, location pages)
- **P2**: Authority-building pages — subtopic and pillar pages
- **P3**: Supporting content — long-tail supporting articles

## Semantic Brief Template

Every page written for this site should follow this brief structure. Apply it to each P1 and P2 page:

```
## Semantic Brief: [Page Title]

**URL:** /[slug]/
**Page type:** [Service / Category / Subtopic / Supporting / Homepage]
**Tier:** [1 / 2 / 3]
**Primary intent:** [Transactional / Commercial / Informational / Navigational]

### Primary entity
[The main entity this page is about]

### Supporting entities to mention
[List of related entities from Phase 2 that should appear naturally on this page]

### Attributes to cover
[From attribute matrix: which popular, prominent, and relevant attributes must this page address?]

### Query path continuity
[What did the user search before landing here? What will they search next?
The page must satisfy the current query AND prepare the user for the next query in their journey.]

### Content structure (H-tags)
H1: [Exact or close-to-exact match of primary intent query]
H2: [Key attribute or service aspect]
H2: [Key attribute or service aspect]
H2: [FAQ or related question from PAA]
H2: [Local signal / trust section]

### Word count target
[X–Y words]

### Internal links from this page
- Link to: [page] using anchor: "[anchor text]"
- Link to: [page] using anchor: "[anchor text]"

### Internal links to this page
- From: [page] using anchor: "[anchor text]"

### Schema markup
[Which schema type: LocalBusiness / Service / FAQPage / Article / etc.]

### Local signals required
[NAP mention, city/neighborhood references, local landmarks if relevant, GBP service match]

### What NOT to include
[Topics outside topical boundary, competitor mentions, off-topic content]
```

## First 3 Priority Briefs

Write fully completed semantic briefs (using the template above) for the 3 highest-priority pages from the content inventory. These should be the homepage and the top 2 service pages.

Fill every field specifically for this business — no generic placeholders.

## Query Path Continuity

One of the most important concepts in Koray's system. Every page must be aware of:
- What query brought the user to THIS page
- What query they will type NEXT if this page doesn't fully satisfy them

Map the query path for the top 5 most valuable pages:

| Page | Previous query (user came from) | Current query (landing intent) | Next query (if not satisfied) | How this page addresses the "next query" |
|------|--------------------------------|-------------------------------|-------------------------------|------------------------------------------|

## Internal Linking Map

Concrete implementation of the link architecture from Phase 3. For each page, list:
- Which pages link TO it (inbound internal links)
- Which pages it links OUT TO (outbound internal links)
- Anchor text for each link

Use the entity relationship map to choose anchor text — anchor text should reflect what the destination page IS about (the entity), not just generic phrases like "click here" or "learn more."

## Content Pruning Decision Log

If the site already has existing content, or if during brief creation any planned page seems redundant, log it:

| Page | Status | Reason | Action |
|------|--------|--------|--------|
| [page] | Thin | < 300 words, no entity coverage | Expand or consolidate into parent page |
| [page] | Duplicate | Same intent as [other page] | Redirect to [other page] |
| [page] | Off-topic | Outside topical boundary | Noindex or delete |

## Output format

Write `04-content-system.md` with H2 sections: Content Inventory Table, Semantic Brief Template, Top 3 Priority Briefs (fully written out), Query Path Map, Internal Linking Map, Content Pruning Log.
