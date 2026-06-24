---
name: workflow
description: Full end-to-end Semantic SEO pipeline for local business websites following Koray Tuğberk Gübür's framework. Runs 5 research phases then site launch — Foundation & Strategy → Semantic Research → Topical Authority → Content System → Technical & Local SEO → Site Launch (hosting, WordPress, GBP, GA4, GSC). Use when the user says "workflow", "/workflow", "build my SEO site", "start the framework", "run the pipeline", "semantic SEO pipeline", "start from scratch", "full Koray framework", or "build a local business site with Semantic SEO".
tools:
  - Read
  - Write
  - Bash
  - WebSearch
---

# Semantic SEO Builder — /workflow

This is the master pipeline skill. It guides the user through 5 research/strategy phases followed by site launch. The research phases must complete first — they produce the blueprint. Only after Phase 5 is the blueprint used to build and launch the actual site.

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

Tell the user: "Phase 5 complete — technical & local SEO plan saved."

---

### Phase 6 — Site Launch

**Only reached after all 5 research phases are complete.**

Summarize what was produced:

```
Blueprint complete:
  01-foundation.md      — market segments, ICP profiles, search intent map
  02-semantic-research.md — entity matrices, attribute research
  03-topical-map.md     — hub-and-spoke architecture, content hierarchy
  04-content-system.md  — content inventory, semantic briefs, internal link map
  05-technical-local.md — site architecture, schema plan, GBP alignment
```

Tell the user: "Research phases complete. Now launching the actual site using this blueprint."

**Automatically trigger `/one-time-setup`** — do not wait for the user to ask. Say: "Starting Phase 6 — Site Launch..." and proceed through the one-time-setup skill. It reads `05-technical-local.md` and `04-content-system.md` to pre-fill business-specific details into every checklist item.

Phase 6 covers:
- Hostinger account + domain registration
- WordPress installation and core settings
- Theme (GeneratePress) + essential plugins
- Rank Math SEO configuration
- Google Business Profile creation and full optimization
- GA4 + Google Search Console connection
- Launch indexing — sitemap submission + P1 page indexing requests

After one-time-setup completes, ask: "Do you want me to write the content for your P1 service pages now? I can write the full page copy for each pillar page using the semantic briefs from Phase 4."

## Rules

- Never skip a phase. Each phase output feeds into the next.
- Always produce the deliverable file before moving to the next phase. Do not announce completion without the file existing.
- Write all files in clean markdown with clear headers.
- Base all research on the actual business niche provided — do not use generic placeholders.
- When performing entity or topic research, use WebSearch to validate against real SERPs.
- Attribute research must look at what real top-ranking pages in the niche actually cover — not generic SEO theory.
