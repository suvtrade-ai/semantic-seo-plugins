# Phase 3 — Topical Authority Building

## Goal
Design the complete topical structure of the site. Topical authority is not about writing more content — it is about writing the RIGHT content with the RIGHT depth in the RIGHT hierarchy. A site with 20 perfectly structured pages outranks a site with 200 unfocused pages.

## Raw Topical Map

Start by brainstorming every topic and subtopic the site could potentially cover within the topical boundary defined in Phase 2. Do not organize yet — just list everything.

Format: flat numbered list of all topics

Example for a custom tailor:
1. Bespoke suits
2. Made-to-measure suits
3. Suit fabrics
4. Wool types for suits
5. Suit alterations
6. Wedding suits
7. Business suits
8. Suit fitting process
9. Lapel styles
10. Suit lining options
... (continue until exhaustive)

Aim for 40–80 topics minimum. Use WebSearch to find topics competitors miss.

## Processed Topical Map

Now organize the raw list into a 3-tier hierarchy:

**Tier 1 — Pillars** (3–6 pillars max)
Each pillar is a major theme area the site will be authoritative about. Each pillar gets its own category/archive page or a very comprehensive service page.

**Tier 2 — Subtopics** (3–8 per pillar)
Each subtopic is a specific angle within the pillar. Each subtopic gets its own page.

**Tier 3 — Supporting content** (1–4 per subtopic)
Detailed supporting articles or FAQ pages that answer specific questions within the subtopic.

Present as indented outline:

```
Pillar: [Pillar Name]
  Subtopic: [Subtopic Name]
    Supporting: [Article Title]
    Supporting: [Article Title]
  Subtopic: [Subtopic Name]
    Supporting: [Article Title]
```

## Topical Depth Table

For each pillar, define the depth of coverage:

| Pillar | Subtopic Count | Supporting Articles Count | Total Pages | Depth Rating |
|--------|---------------|--------------------------|-------------|--------------|
| Pillar A | 4 | 12 | 17 | Deep |
| Pillar B | 3 | 6 | 10 | Medium |

Depth ratings:
- **Deep** (primary revenue pillar): exhaustive coverage, 4+ subtopics, 3+ supporting articles each
- **Medium** (secondary pillar): solid coverage, 3–4 subtopics, 1–2 supporting articles each
- **Shallow** (supporting pillar): surface coverage, 2 subtopics, 1 supporting article each. Only add shallow pillars if they are genuinely relevant to the entity map.

## Content Pruning Logic

Not everything in the raw topical map belongs on this site. Apply pruning rules:

**Prune if:**
- The topic sits outside the topical boundary (Phase 2 context vectors)
- The topic has zero search demand for this geography/niche
- The topic would make the site seem like a general informational site rather than a specialized local business
- A competitor page ranking for this topic has 1,000+ words of genuinely superior content and there is no differentiation angle

**Keep if:**
- The topic answers a question Google associates with the primary entity (check PAA boxes)
- The topic appears in the entity relationship map from Phase 2
- Covering the topic demonstrates expertise that converts visitors to customers

Log every pruned topic with the reason: Topic | Pruned Reason | Alternative Treatment (redirect, brief mention on parent page, skip entirely)

## Internal Link Architecture

Internal links in the Koray framework are NOT random. They follow topical relationship logic:

**Rules:**
1. Pillar pages link DOWN to all their subtopic pages (hub → spoke)
2. Subtopic pages link UP to their pillar (spoke → hub)
3. Subtopic pages link LATERALLY to sibling subtopics within the same pillar only
4. Supporting articles link UP to their subtopic page only
5. The homepage links to all pillar pages
6. Service pages are treated as Tier 1 (pillar) pages for commercial intent

Present as a link flow diagram (text-based):

```
Homepage
  → Pillar A page
      → Subtopic A1 page ← → Subtopic A2 page
          → Supporting A1a
          → Supporting A1b
  → Pillar B page
      → Subtopic B1 page
```

Also produce a link matrix table: Source Page | Target Page | Anchor Text Suggestion | Link Type (nav / inline / footer)

## Output format

Write `03-topical-map.md` with H2 sections: Raw Topical Map, Processed Topical Map, Topical Depth Table, Content Pruning Log, Internal Link Architecture.
