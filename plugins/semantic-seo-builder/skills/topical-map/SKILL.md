---
name: topical-map
description: Runs Phase 3 of the Semantic SEO framework — Topical Authority Building. Uses SerpAPI keyword expansion and SERP overlap validation to build a 3-tier hierarchy (pillar, subtopic, supporting), content pruning log, and internal link architecture. Produces 03-topical-map.md. Use when user says topical map, topical authority, or start Phase 3.
tools:
  - Write
  - WebSearch
  - Bash
---

# Topical Authority Building (Phase 3)

Read `02-semantic-research.md` if it exists for entity and context data. Read `01-foundation.md` for business context. Otherwise ask for business type, services, city, and topical boundary.

Then produce `03-topical-map.md` containing:

## Step 0 — SerpAPI Keyword Expansion

Before building the topical map, use SerpAPI to pull real keyword signals for each pillar theme. This grounds the map in actual search demand rather than editorial guesswork.

```python
import os, requests, json

SERPAPI_KEY = os.environ.get("SERPAPI_API_KEY")

# Read pillar themes from 02-semantic-research.md entity list
# Define your pillar seed keywords here
pillar_seeds = [
    f"{service} {city}",           # e.g. "custom tailor Bangkok"
    f"bespoke {service} {city}",
    f"{service} price {city}",
    # add one per pillar theme
]

serp_data = {}
for seed in pillar_seeds:
    params = {
        "engine": "google",
        "q": seed,
        "api_key": SERPAPI_KEY,
        "num": 10,
        "gl": "th",
        "hl": "en"
    }
    resp = requests.get("https://serpapi.com/search", params=params).json()
    serp_data[seed] = {
        "paa":     [q["question"] for q in resp.get("related_questions", [])],
        "related": [s["query"] for s in resp.get("related_searches", [])],
        "top_urls": [r["link"] for r in resp.get("organic_results", [])[:10]],
        "has_map":  len(resp.get("local_results", {}).get("places", [])) > 0,
    }
    print(f"\n{seed}: PAA={serp_data[seed]['paa']}, Related={serp_data[seed]['related']}")

with open("serp-keyword-map.json", "w") as f:
    json.dump(serp_data, f, indent=2)
print("Saved serp-keyword-map.json")
```

**Use `serp-keyword-map.json` to:**
- Expand Tier 3 supporting articles from PAA questions (each PAA = one supporting article)
- Validate Tier 2 subtopics against related searches (if a related search has no subtopic, add one)
- Flag pillars where `has_map=True` — those need dedicated local SEO treatment in Phase 5
- Detect SERP overlap: if two pillar seeds share 4+ of the same `top_urls`, merge them into one pillar

### SERP Overlap Validation

For any two pillar keywords you're considering as separate pages, run this overlap check:

```python
from itertools import combinations

def overlap_count(urls_a, urls_b):
    return len(set(urls_a) & set(urls_b))

pairs = list(combinations(pillar_seeds, 2))
for a, b in pairs:
    overlap = overlap_count(serp_data[a]["top_urls"], serp_data[b]["top_urls"])
    verdict = "MERGE" if overlap >= 4 else "KEEP SEPARATE"
    print(f"{a} vs {b}: {overlap} shared URLs → {verdict}")
```

**Rule:** ≥4 shared top-10 results = Google treats these as the same topic → merge into one page. <4 = distinct topics → separate pages.

---

## Raw Topical Map
Brainstorm 40–80 topics and subtopics within the topical boundary. Seed from `serp-keyword-map.json` (PAA + related searches) and expand with WebSearch for topics competitors miss. Flat numbered list.

## Processed Topical Map
Organize into 3-tier hierarchy:
- Tier 1 — Pillars (3–6): major theme areas, one page each
- Tier 2 — Subtopics (3–8 per pillar): specific angles, one page each
- Tier 3 — Supporting (1–4 per subtopic): detailed articles or FAQ pages

Present as indented outline.

## Topical Depth Table
Table: Pillar | Subtopic Count | Supporting Article Count | Total Pages | Depth Rating (Deep/Medium/Shallow)

Primary revenue pillars = Deep. Secondary = Medium. P