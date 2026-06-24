---
name: content-brief
description: Runs Phase 4 of the Semantic SEO framework — Content System. Builds a content inventory, creates semantic content briefs (not keyword briefs) for each page, maps query path continuity, and produces an internal linking map. Use when the user says "content brief", "/content-brief", "semantic brief", "content inventory", "write a brief", "brief for my pages", "content plan", "query path", "what should my page cover", or "start Phase 4". Also use when the user names a specific page and asks for a semantic brief for it.
tools:
  - Write
  - WebSearch
  - Read
---

# Content System (Phase 4)

Read `03-topical-map.md` for the content inventory source. Read `02-semantic-research.md` for entities and attributes. If these files don't exist, ask the user to provide their list of planned pages and their entity research.

If the user asks for a brief for ONE specific page (not the full pipeline), jump directly to writing a single semantic brief using the template below. Ask: what is the page about, what is the URL slug, and what is the primary intent?

## Full Phase 4 Output (when running the full pipeline)

Produce `04-content-system.md` containing:

### Content Inventory Table
Every planned page as a row:
# | Page Title | URL Slug | Page Type | Tier | Primary Entity | Target Intent | Word Count | Priority (P1/P2/P3)

P1 = revenue pages (homepage, service pages, location pages)
P2 = authority pages (pillar, subtopic)
P3 = supporting content (long-tail articles)

### Semantic Brief Template
The standard brief template to be used for every page on the site. Include all fields from the structure below.

### Top 3 Priority Briefs
Write fully completed semantic briefs for the top 3 P1 pages. Every field must be filled with business-specific content — no placeholders.

**Semantic brief structure:**

```
## Semantic Brief: [Page Title]

URL: /[slug]/
Page type: [Service / Category / Subtopic / Supporting / Homepage]
Tier: [1 / 2 / 3]
Primary intent: [Transactional / Commercial / Informational / Navigational]

Primary entity: [main entity this page targets]

Supporting entities to mention:
- [entity 1]
- [entity 2]
- [entity 3]

Attributes to cover:
- Popular: [list]
- Prominent: [list]
- Relevant (differentiation): [list]

Query path continuity:
- User searched before landing: [query]
- Current page satisfies: [query]
- User will search next if not satisfied: [query]
- How this page addresses the next query: [specific section or content element]

Content structure (H-tags):
H1: [primary query match]
H2: [key attribute/service aspect]
H2: [key attribute/service aspect]
H2: [FAQ / PAA question]
H2: [Local signal / trust section]

Word count target: [X–Y words]

Internal links OUT:
- To: [page] | Anchor: "[text]"

Internal links IN:
- From: [page] | Anchor: "[text]"

Schema: [type]

Local signals: [NAP placement, city/neighborhood mentions, landmarks]

Do NOT include: [off-topic content, out-of-boundary topics]
```

### Query Path Map
Table for top 5 pages: Page | Previous query | Current query | Next query | How this page bridges to next

### Internal Linking Map
Concrete link matrix: Source Page | Target Page | Anchor Text | Link Type

### Content Pruning Log
Table: Page | Status | Reason | Action (expand / consolidate / redirect / delete / noindex)

Confirm `04-content-system.md` was saved.

---

## Step 5 — Write the full page (optional, P1 and P2 pages)

After producing the semantic brief, offer to write the full page content immediately:

"Brief is ready. Want me to write the full page now? I'll follow the brief exactly and produce copy ready to paste into WordPress."

If yes, write the complete page in clean markdown:

**For P1 service pages (e.g. Bespoke Suits, Alterations):**
- H1: matches primary transactional query
- Opening paragraph (100 words): answer-first — what the service is, who it's for, where it is
- H2 sections: one per prominent attribute from the attribute matrix
- Process H2: step-by-step how the service works (3–5 steps)
- Fabric/material H2 (if applicable): cover fabric entities and attributes
- FAQ H2: 4–6 PAA-style questions answered in 50–80 words each
- Closing CTA paragraph: link to booking/WhatsApp, mention location

**For P2 subtopic pages:**
- H1: matches informational or commercial intent query
- Answer-first opening (80 words)
- H2 sections: cover popular → prominent → relevant attributes in that order
- 1–2 internal links to P1 parent page using entity-based anchor text
- Closing sentence linking to relevant service page

**Word counts:**
- P1 pages: 800–1,200 words (service pages, not essays)
- P2 pages: 600–900 words
- Supporting blog: 800–1,500 words (handled by /weekly-pipeline)

**After writing:**
- Provide the content in a code block ready to paste into WordPress
- Suggest the URL slug
- Note which schema type to add via Rank Math (Service, LocalBusiness, FAQPage)
- Suggest the featured image description for `/image-gen`
- Remind user to add internal links from related pages TO this new page



---

## Integration: claude-blog skills (Phase 4 content production)

Phase 4 produces briefs AND content. Use these claude-blog skills in sequence for each page:

### Step 1 — Brief (blog-brief)

```
/blog brief [page topic] --from cluster-plan.json
```

`blog-brief` generates a full competitive brief: SERP analysis, competitor coverage gaps, keyword density guidance, recommended statistics, image and chart suggestions, and section-by-section word count targets. This is the production brief that writers (or blog-write) work from.

**Feed into this brief:** the entity and attribute lists from `02-semantic-research.md` and the page's position in the topical map from `03-topical-map.md`. The brief should explicitly name which Koray attributes each H2 section must cover.

### Step 2 — Write (blog-write)

```
/blog write [page topic] --from brief
```

`blog-write` produces the full article from the brief: answer-first formatting, Key Takeaways box, sourced statistics, FAQ schema, internal link zones, image placeholders, and proper heading hierarchy. For MDX/WordPress output, it handles frontmatter and schema JSON-LD.

**After writing:** the article goes immediately into the fact-check and SEO check pipeline below before saving to the content system.

### Step 3 — Fact-check (blog-factcheck)

```
/blog factcheck [article file]
```

Verifies every statistic and claim against its cited source URL. Returns a confidence score per claim (1.0 = exact match, 0.7-0.9 = paraphrase, 0.0 = not found on source page). Fix any 0.0-scored claims before publishing — either find the real source or remove the claim.

**If Kie API is connected:** for claims where the original source is behind a paywall or has been removed, generate an equivalent statistic via Kie's research query capability rather than fabricating one.

### Step 4 — Schema (blog-schema)

```
/blog schema [article file]
```

Generates complete JSON-LD schema for the article: BlogPosting (or Article), Person/Organization author, BreadcrumbList, FAQPage (from the article's FAQ section), and ImageObject. Output is ready to paste into Rank Math's custom schema field.

**For local business pages (not blog posts):** use `seo-schema` instead — it handles LocalBusiness, Service, and Review schema types that blog-schema doesn't cover.

### Step 5 — GEO optimization (blog-geo)

```
/blog geo [article file]
```

Audits the article for AI citation readiness — scores passage-level citability, Q&A formatting, entity clarity, and structured data. Generates citation capsules: self-contained answer paragraphs that AI systems (ChatGPT, Perplexity, Google AI Overviews) can lift verbatim.

Target: AI Citation Readiness score > 70. Articles below 60 need rewriting before publication.

### Step 6 — Images (image-gen)

```
/image-gen [article file]
```

Our Kie API image generation skill reads the article's section structure and generates section-appropriate images: hero, service illustration, process diagram, fabric/material detail. Converts to WebP at 72% quality (target under 100KB per image).

**Do NOT use `blog-image` or `seo-image-gen`** — we have the Kie API image-gen skill which is configured for this pipeline.

---

### Complete Phase 4 sequence per page:

1. `/blog brief [topic]` → produces brief
2. `/blog write [topic] --from brief` → produces article
3. `/blog factcheck [article]` → verifies claims
4. `/blog schema [article]` → generates JSON-LD
5. `/blog geo [article]` → checks AI citability
6. `/image-gen [article]` → generates + places images
7. Save final article to `04-content-system.md` inventory

Write each completed article URL to `weekly-log.md` for the monthly audit to track.
