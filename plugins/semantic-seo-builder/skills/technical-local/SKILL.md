---
name: technical-local
description: Runs Phase 5 of the Semantic SEO framework — Technical & Local SEO Layer. Designs site architecture, URL structure, internal link flow, crawl settings, schema markup plan, local SEO signal map, GBP optimization checklist, and WordPress implementation guide. Produces 05-technical-local.md. Use when user says technical SEO, site architecture, or start Phase 5.
tools:
  - Write
  - Read
---

# Technical & Local SEO Layer (Phase 5)

Read `03-topical-map.md` for the site structure and page hierarchy. Read `02-semantic-research.md` for entity and location data. If neither exists, ask for: list of planned pages, business address, phone number, and platform (WordPress assumed).

Produce `05-technical-local.md` containing:

## Site Architecture
Text-based hierarchy diagram showing the full URL tree from homepage down to supporting content. Every page from the topical map must appear here in the correct tier.

## URL Structure Rules
Table: Page Type | URL Pattern | Example
Rules: lowercase, hyphens, no stop words, entity name in slug, 3–5 words max.

## Internal Link Flow Rules
Six rules (enumerate and enforce):
1. Nav menu: pillar pages only
2. Footer: all pillars + About + Contact
3. Breadcrumbs: all pages except homepage (implement via Rank Math / Yoast)
4. In-content links: follow Phase 3 link matrix exactly
5. Sidebar (if used): same-pillar subtopics only
6. No orphan pages: every page must have ≥ 2 inbound internal links

## Crawl Efficiency Checklist
Checklist of robots.txt rules, sitemap settings, noindex rules, pagination, canonical settings. Each item is a checkbox with clear implementation instruction.

## Schema Markup Plan
Table: Page Type | Schema @type | Key Properties to Include
Critical: LocalBusiness on homepage must use the most specific @type, include sameAs array (social + GBP URL), geo coordinates, hasMap link.

### floorLevel Property (mandatory for multi-story buildings)

If the business is located inside a multi-story building (mall, office tower, plaza, shared commercial building), add the `floorLevel` property to the LocalBusiness schema. This is a Google-introduced schema.org property that disambiguates businesses sharing the same building address — directly relevant for Google Maps and local pack accuracy.

**Critical rule:** `floorLevel` is a property of `LocalBusiness` (and its subtypes), NOT a property of `PostalAddress`. It must sit OUTSIDE the `address` wrapper.

```json
{
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "@id": "https://yourdomain.com/#localbusiness",
  "name": "Business Name",
  "url": "https://yourdomain.com/",
  "telephone": "+66-XX-XXX-XXXX",
  "address": {
    "@type": "PostalAddress",
    "streetAddress": "419/A Oxford 5, Platinum Fashion Mall",
    "addressLocality": "Bangkok",
    "addressRegion": "Bangkok",
    "postalCode": "10400",
    "addressCountry": "TH"
  },
  "floorLevel": "2nd Floor",
  "geo": {
    "@type": "GeoCoordinates",
    "latitude": "XX.XXXXX",
    "longitude": "XX.XXXXX"
  }
}
```

Use the floor level value confirmed in `01-foundation.md` (gathered during Phase 1 intake). Examples: `"2nd Floor"`, `"5th Floor"`, `"Ground Floor"`, `"Level 3"`.

Check: if `01-foundation.md` records a multi-story building address but `floorLevel` is missing from the schema — flag as a critical schema gap.

## Local SEO Signal Map
NAP consistency rule: where the name/address/phone must appear identically (footer, contact page, about page, schema, GBP, citations).

GBP alignment:
- GBP primary category → must match schema @type
- GBP services → must match service entities from Phase 2
- GBP description → must contain top 5 Phase 2 entities
- GBP posts → link to Phase 4 content
- GBP service area → matches Phase 2 location entities

## GBP Optimization Checklist
Checkbox list covering: categories (primary + secondary), services with descriptions, products if applicable, business description (750 chars), photos minimum (10), Q&A pre-seeding (5 questions), posts schedule, review request workflow, messaging.

## WordPress Technical Implementation
- Theme recommendation and reason
- Essential plugin list with purpose for each
- Permalink structure setting
- Core Web Vitals targets (LCP, CLS, INP)
- Rank Math / Yoast entity configuration steps (Knowledge Graph settings)

Confirm `05-technical-local.md` was saved.


---

## Integration: claude-seo skills (Phase 5 implementation and validation)

After producing `05-technical-local.md`, run these skills to validate and implement:

### seo-technical — full technical audit

```
/seo technical [domain]
```

Runs a 9-category technical audit: crawlability, indexability, security, URL structure, mobile, Core Web Vitals, structured data, JavaScript rendering, and IndexNow. Run this after WordPress setup is complete and the theme is live (before content production begins).

**Target scores for a new local business site:**
- Crawlability: 90+
- Mobile: 95+ (mobile-first indexing is 100% complete since July 2024)
- Core Web Vitals: LCP < 2.5s, INP < 200ms, CLS < 0.1
- Security: 85+ (HTTPS + basic security headers)

Flag any category below 80 as a blocker before content production.

### seo-sitemap — generate and validate sitemap

```
/seo sitemap [domain]
```

Generates the XML sitemap from the URL structure defined in `05-technical-local.md`, validates format, and confirms it's referenced in robots.txt. Submit the output URL to Google Search Console via the indexing skill after generation.

### seo-local — GBP and local signals

```
/seo local [business name] [city]
```

Validates the full local SEO signal map: NAP consistency across all touchpoints, GBP category alignment with schema @type, citation presence on key directories, review signals, and location page quality. Cross-references against the GBP optimization checklist in `05-technical-local.md`.

**Run this after:** GBP is created and connected to the website (Step 7 of one-time-setup).

### seo-schema — validate LocalBusiness schema

```
/seo schema [homepage URL]
```

Validates the full LocalBusiness JSON-LD including the `floorLevel` property (if applicable), `sameAs` array completeness, geo coordinates, and hasMap link. Flags any properties that fail Google's Rich Results Test requirements.

**Critical check:** confirm `floorLevel` sits OUTSIDE the `address` PostalAddress wrapper. This is the most common implementation error for multi-story business schema.

### seo-images — image optimization audit

```
/seo images [domain]
```

Checks all images on P1 pages: alt text presence and quality, file sizes (flag > 200KB), format compliance (WebP preferred), responsive image attributes, lazy loading implementation, and CLS prevention (explicit width/height). Run after all Kie API images are placed and before site launch.
