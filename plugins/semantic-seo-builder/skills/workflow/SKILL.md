---
name: workflow
description: Full end-to-end Semantic SEO pipeline for local business websites following Koray Tuğberk Gübür's framework. 7 phases: Foundation → Semantic Research → Topical Authority → Content System → Technical & Local SEO → Static HTML Pages (responsive, mobile, images) → Site Launch (hosting, WordPress, GBP, GA4, GSC). Use when the user says "workflow", "/workflow", "build my SEO site", "start the framework", "run the pipeline", "semantic SEO pipeline", "start from scratch", "full Koray framework", or "build a local business site with Semantic SEO".
tools:
  - Read
  - Write
  - Bash
  - WebSearch
---

# Semantic SEO Builder — /workflow

This is the master pipeline skill. It guides the user through 7 phases: 5 research phases that produce the blueprint, then Phase 6 which builds static HTML preview pages from that blueprint (with images), then Phase 7 which launches the live site on WordPress.

## How to run the pipeline

### Step 0 — Gather business context

Before starting any phase, ask the user these intake questions (use AskUserQuestion if available, otherwise ask in plain text):

1. What is the business? (name, niche, services)
2. What city/area does it serve?
3. Who are the 2–3 main competitors?
4. Does a website already exist, or is this a new build?
5. What platform will the site run on? (WordPress recommended)

Store answers as variables and reference them throughout all phases.

---

### Phase 1 — Foundation & Strategy

Read `references/phase-1-foundation.md` for full instructions.

Produce file: `01-foundation.md`

The file must contain:
- Business overview (niche, services, geography)
- Market segmentation (4–6 segments)
- ICP profiles (2–3 ideal customer profiles with search behavior)
- Search intent map (commercial, transactional, navigational, informational queries for this business)
- Competitor gap table (semantic gaps, topical gaps, entity gaps vs. each competitor)

Tell the user: "Phase 1 complete — foundation document saved. Moving to Phase 2."

---

### Phase 2 — Semantic Research

Read `references/phase-2-semantic-research.md` for full instructions.

Produce file: `02-semantic-research.md`

The file must contain:
- Primary entities (business type, services, locations, people, tools)
- Entity validation table (source: Wikipedia, Google Knowledge Panel, SERP entities, schema.org)
- Attribute matrix:
  - Popular attributes (what most pages in this niche cover)
  - Prominent attributes (what top-ranked pages emphasize)
  - Relevant attributes (semantically related but underused)
- Entity relationship map (how entities connect to each other)
- Context vectors (what topics form the topical boundary for this niche)

Tell the user: "Phase 2 complete — semantic research saved. Moving to Phase 3."

---

### Phase 3 — Topical Authority Building

Read `references/phase-3-topical-authority.md` for full instructions.

Produce file: `03-topical-map.md`

The file must contain:
- Raw topical map (flat list of all topics and subtopics this site should cover)
- Processed topical map (organized into hierarchy with depth levels)
- Pillar → Subtopic → Supporting content structure (3-tier hierarchy)
- Topical depth table (how many supporting articles per subtopic)
- Content pruning notes (topics to exclude and why — avoid dilution)
- Internal link architecture (which pages link to which, based on topical relationships)

Tell the user: "Phase 3 complete — topical map saved. Moving to Phase 4."

---

### Phase 4 — Content System

Read `references/phase-4-content-system.md` for full instructions.

Produce file: `04-content-system.md`

The file must contain:
- Content inventory table (all planned pages with: URL slug, page type, pillar/subtopic/supporting, primary entity, target intent, word count estimate)
- Semantic brief template (reusable template for writing each page)
- First 3 fully written semantic content briefs for the highest-priority pages
- Internal linking map (anchor text, source page, destination page)
- Content pruning decision log (any existing content marked for removal or consolidation)

Tell the user: "Phase 4 complete — content system saved. Moving to Phase 5."

---

### Phase 5 — Technical & Local SEO Layer

Read `references/phase-5-technical-local.md` for full instructions.

Produce file: `05-technical-local.md`

The file must contain:
- Site architecture diagram (text-based: homepage → category pages → service pages → supporting pages)
- URL structure rules (pattern for each page type)
- Internal link flow rules (top-down + horizontal linking logic)
- Crawl efficiency checklist (robots.txt, sitemap, pagination, noindex rules)
- Schema markup plan (which schema types go on which page types)
- Local SEO signal map (NAP consistency, GBP categories, GBP services vs. site entities)
- GBP alignment checklist (GBP description entities must match homepage and service page entities)
- Technical implementation checklist (WordPress-specific: theme, plugins, permalink structure, page speed)

Tell the user: "Phase 5 complete — technical & local SEO plan saved. Moving to Phase 6."

---

### Phase 6 — Static HTML Pages

**Only reached after all 5 research phases are complete.**

Summarize what was produced:

```
Blueprint complete:
  01-foundation.md        — market segments, ICP profiles, search intent map
  02-semantic-research.md — entity matrices, attribute research
  03-topical-map.md       — hub-and-spoke architecture, content hierarchy
  04-content-system.md    — content inventory, semantic briefs, internal link map
  05-technical-local.md   — site architecture, schema plan, GBP alignment
```

Tell the user: "Blueprint complete. Building static HTML preview pages now."

**Automatically trigger `/html-pages`** — do not wait for the user to ask. Say: "Starting Phase 6 — HTML Page Builder..." and run the html-pages skill. It reads all 5 phase documents and builds every page in the content inventory as responsive, mobile-optimised HTML with images.

Phase 6 produces:
- `preview/index.html` — Homepage with full sections and hero image
- `preview/services/[slug].html` — One file per pillar service with service-specific images
- `preview/about.html` — About / story page with workshop images
- `preview/contact.html` — Contact page with click-to-call and Google Maps embed
- `preview/blog/index.html` — Blog listing page
- `preview/css/style.css` — Shared responsive stylesheet
- `preview/js/main.js` — Hamburger nav toggle
- `preview/images/` — All WebP images, each under 100KB

Every page is mobile-optimised: hamburger nav, clamp() typography, responsive grids, click-to-call phone numbers, no fixed heights.

Tell the user: "Phase 6 complete — all pages built with images. Moving to Phase 7."

---

### Phase 7 — Site Launch

**Only reached after Phase 6 HTML pages are complete.**

Tell the user: "HTML pages are ready. Now setting up the live site."

**Automatically trigger `/one-time-setup`** — do not wait for the user to ask. Say: "Starting Phase 7 — Site Launch..." and proceed through the one-time-setu