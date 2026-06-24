---
name: topical-map
description: Runs Phase 3 of the Semantic SEO framework — Topical Authority Building using Koray's methodology. Creates a raw topical map, converts it to a processed hierarchy (pillar → subtopic → supporting), defines topical depth, applies content pruning logic, and designs the internal link architecture. Produces 03-topical-map.md. Use when the user says "topical map", "/topical-map", "topical authority", "content hierarchy", "pillar content", "hub and spoke", "content structure", "build my topical map", "Koray topical map", or "start Phase 3".
tools:
  - Write
  - WebSearch
---

# Topical Authority Building (Phase 3)

Read `02-semantic-research.md` if it exists for entity and context data. Read `01-foundation.md` for business context. Otherwise ask for business type, services, city, and topical boundary.

Then produce `03-topical-map.md` containing:

## Raw Topical Map
Brainstorm 40–80 topics and subtopics within the topical boundary. Use WebSearch to discover topics competitors miss. Flat numbered list.

## Processed Topical Map
Organize into 3-tier hierarchy:
- Tier 1 — Pillars (3–6): major theme areas, one page each
- Tier 2 — Subtopics (3–8 per pillar): specific angles, one page each
- Tier 3 — Supporting (1–4 per subtopic): detailed articles or FAQ pages

Present as indented outline.

## Topical Depth Table
Table: Pillar | Subtopic Count | Supporting Article Count | Total Pages | Depth Rating (Deep/Medium/Shallow)

Primary revenue pillars = Deep. Secondary = Medium. Peripheral = Shallow.

## Content Pruning Log
Apply pruning rules. Log every pruned topic:
Table: Topic | Pruned Reason | Alternative Treatment

Prune if: outside topical boundary, zero search demand for this niche/geo, would dilute topical focus, no differentiation angle vs. existing top-ranked content.

## Internal Link Architecture
Design following strict rules:
- Pillar pages link DOWN to subtopics
- Subtopic pages link UP to pillar + LATERALLY to sibling subtopics only
- Supporting articles link UP to subtopic only
- Homepage links to all pillars
- Service pages = Tier 1 (pillar)

Produce text-based hierarchy diagram + link matrix table:
Source Page | Target Page | Anchor Text Suggestion | Link Type (nav/inline/footer)

Confirm `03-topical-map.md` was saved.


---

## Integration: seo-cluster (execute the cluster plan)

If `cluster-plan.json` was produced in Phase 2, import it now to anchor the topical map architecture:

```
/seo cluster plan --from strategy
```

This reads the Phase 2 cluster output and enriches it with full SERP overlap analysis. The resulting hub-and-spoke structure becomes the skeleton of `03-topical-map.md`.

**Important:** seo-cluster uses SERP overlap (shared Google top-10 results) to group content — this is objective data, not editorial guesswork. Where the cluster plan and your Koray topical judgment diverge, trust the SERP data for page boundaries and trust the Koray framework for attribute depth within each page.

After Phase 3 is finalized, run:
```
/seo cluster map
```
This generates `cluster-map.html` — an interactive visualization of the full topical architecture. Share it with the client for alignment before content production begins.

---

## Integration: seo-sxo (intent validation)

Before finalizing the topical map, validate the top 3 pillar pages using the Search Experience Optimization skill:

```
/seo sxo [pillar keyword]
```

`seo-sxo` reads the Google SERP backwards — it analyzes what page types, content formats, and user intents Google is actually rewarding for each pillar keyword. If Google rewards a "best-of" list but you planned a service page, the service page won't rank regardless of entity coverage depth.

**Use the sxo output to:**
- Confirm or adjust the content format (guide vs. list vs. comparison vs. service page)
- Validate the intended word count against what's ranking
- Identify any SERP feature opportunities (featured snippets, PAA, local pack) that should influence page structure

Add the sxo findings to `03-topical-map.md` under each pillar's entry as "SERP intent signal."
