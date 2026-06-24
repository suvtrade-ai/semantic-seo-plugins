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
Identify entities across these types: Business entity, Service entities, Location entities, Credential entities, Tool/material entities, Process entities, Person entities, Problem entities.

List all as a table: Entity | Type | Relevance | Source

## Entity Validation
For each entity, validate via:
- Source-based: Wikipedia, schema.org, Google Knowledge Panel
- SERP-based: Knowledge Panel presence, consistent competitor mentions, PAA coverage
- Contextual: does removing the entity make the content feel incomplete?

Mark each: Validated | Partially Validated | Excluded (reason)

## Attribute Matrix
Search top 20 pages for the primary query. Extract:
- Popular attributes: what most pages mention
- Prominent attributes: what TOP 3 pages emphasize heavily
- Relevant attributes: semantically adjacent but underused

Build table: Entity | Popular | Prominent | Relevant | Coverage Gap

## Entity Relationship Map
Map how entities relate: is-a, has-a, used-for, located-in, associated-with, part-of
Table: Entity A | Relationship | Entity B | Importance

## Context Vectors & Topical Boundary
List: In-scope topics (site should cover) vs. Out-of-scope topics (would dilute authority).
Write 5–10 context signal sentences describing the topical space.

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
