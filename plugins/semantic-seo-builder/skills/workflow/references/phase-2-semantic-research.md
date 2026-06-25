# Phase 2 — Semantic Research

## Goal
Identify, validate, and map the entities, attributes, and relationships that define the topical space. Google understands the web as a graph of entities — this phase maps that graph for the business.

## Entity Research (Koray Methodology)

An entity is any named concept Google can identify and connect to the Knowledge Graph: people, places, organizations, services, tools, certifications, processes.

### Primary entity identification

Start with the business itself as the root entity. Expand outward:

1. **Business entity**: the business type (e.g., "custom tailor", "plumber", "personal injury attorney")
2. **Service entities**: each distinct service offered
3. **Location entities**: city, neighborhood, landmark proximity, service area
4. **Credential entities**: certifications, associations, awards, training programs
5. **Tool/material entities**: specific products, materials, equipment used
6. **Process entities**: methods, techniques, procedures unique to this niche
7. **Person entities**: owner, specialists, named professionals (if relevant)
8. **Problem entities**: the problems customers bring (e.g., "burst pipe", "ill-fitting suit", "whiplash injury")

List all entities in a flat table: Entity | Type | Relevance to Business | Source (KG / Wikipedia / SERP)

## Entity Validation

Validate each entity using three methods:

**Source-based validation**
- Does Wikipedia have an article for this entity?
- Does schema.org have a type for it?
- Does Google's Knowledge Graph return a panel for it?

**SERP-based validation**
- Search the entity name. Does Google return a Knowledge Panel?
- Do top-ranking pages in the niche consistently mention this entity?
- Does the "People also ask" section surface questions about this entity?

**Contextual validation**
- Does removing this entity from content make the content feel incomplete to a knowledgeable reader?
- Does this entity appear in industry publications, trade associations, or regulatory bodies?

Mark each entity: Validated | Partially Validated | Excluded (reason)

## Attribute Research

Attributes are the properties and characteristics associated with each entity. They are what Google expects to see covered around an entity to consider a page semantically complete.

### Popular attributes
What attributes does the average page in this niche mention? (Survey top 20 results for the primary query)
Example for "custom tailor": fabric type, measurement process, turnaround time, price range, alteration types

### Prominent attributes
What attributes appear most heavily on the TOP 3 ranked pages? These are the attributes Google rewards.
Example: fabric sourcing, bespoke vs. made-to-measure distinction, fitting session count

### Relevant attributes
What attributes are semantically adjacent but underused by competitors? These are differentiation opportunities.
Example: suit construction methods (canvas vs. fused), care instructions, seasonal fabric recommendations

Build attribute matrix: Entity | Popular Attributes | Prominent Attributes | Relevant Attributes | Coverage Gap

## Entity Relationship Mapping

Map how entities relate to each other using relationship types:
- **is-a** (a bespoke suit IS-A type of custom garment)
- **has-a** (a tailor HAS-A fitting process)
- **used-for** (wool fabric USED-FOR suiting)
- **located-in** (the business LOCATED-IN the city)
- **associated-with** (Savile Row ASSOCIATED-WITH bespoke tailoring)
- **part-of** (lapel IS-PART-OF a jacket)

Present as: Entity A | Relationship Type | Entity B | Importance (core / supporting / peripheral)

## Context Vectors & Topical Boundary

Context vectors define what Google "expects" to see on a site in this topical space. They prevent topic dilution.

**Topical boundary**: List topics that ARE within scope (the site should cover these) vs. topics that are OUTSIDE scope (covering these dilutes topical authority).

Example — custom tailor site:
- In scope: suit fabrics, alterations, dress codes, wedding attire, business attire, care of garments
- Out of scope: fast fashion reviews, luxury watches, general lifestyle content

**Context signal sentences**: Write 5–10 sentences that describe the topical context the site operates in. These will guide content writers on what is and isn't on-topic.

## Output format

Write `02-semantic-research.md` with H2 sections for: Entity List, Validation Table, Attribute Matrix, Relationship Map, Context Vectors & Topical Boundary.
