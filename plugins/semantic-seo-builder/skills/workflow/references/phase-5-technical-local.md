# Phase 5 — Technical & Local SEO Layer

## Goal
Build the structural foundation that carries topical authority signals to Google efficiently. Technical SEO in this framework is not about generic best practices — every decision must support the topical map and entity graph from Phases 2–4.

## Site Architecture Diagram

Map every planned page into a visual hierarchy (text-based):

```
/ (Homepage)
├── /[pillar-1-slug]/               ← Category/Service pillar page
│   ├── /[pillar-1]/[subtopic-a]/   ← Subtopic page
│   │   └── /[pillar-1]/[subtopic-a]/[supporting-1]/
│   └── /[pillar-1]/[subtopic-b]/
├── /[pillar-2-slug]/
│   ├── /[pillar-2]/[subtopic-a]/
│   └── /[pillar-2]/[subtopic-b]/
├── /about/
├── /contact/
└── /blog/ (if supporting content is used)
    ├── /blog/[supporting-article-1]/
    └── /blog/[supporting-article-2]/
```

Key rules:
- Service pages live at top level or one level deep (not buried at /category/subcategory/service/)
- Supporting content lives under /blog/ or as a subfolder of its subtopic
- The architecture must reflect the topical hierarchy from Phase 3 exactly

## URL Structure Rules

Define the URL pattern for each page type:

| Page Type | URL Pattern | Example |
|-----------|-------------|---------|
| Homepage | / | / |
| Pillar/Service | /[service-slug]/ | /bespoke-suits/ |
| Subtopic | /[pillar]/[subtopic]/ | /bespoke-suits/suit-fabrics/ |
| Supporting article | /blog/[topic-slug]/ | /blog/canvas-vs-fused-construction/ |
| Location page | /[service]-[city]/ | /bespoke-suits-london/ |
| About | /about/ | /about/ |
| Contact | /contact/ | /contact/ |

Rules:
- All lowercase, hyphens only (no underscores, no capital letters)
- No stop words in slugs (no "the", "a", "and", "of")
- Slugs should contain the primary entity name where possible
- Keep slugs short (3–5 words max)

## Internal Link Flow Rules

Enforce these throughout the WordPress build:

1. **Navigation menu**: Pillar pages only (no subtopics in main nav — keeps nav shallow)
2. **Footer links**: All pillar pages + About + Contact
3. **Breadcrumbs**: Required on all pages except homepage (Yoast or Rank Math handles this)
4. **In-content links**: Must follow Phase 3 link matrix exactly
5. **Sidebar links (if used)**: Related subtopics within the same pillar only — never cross-pillar
6. **No orphan pages**: Every page except homepage must have at least 2 inbound internal links

## Crawl Efficiency Checklist

- [ ] robots.txt: disallow /wp-admin/, /wp-login.php, /feed/, /xmlrpc.php
- [ ] robots.txt: allow all service and content pages
- [ ] XML sitemap: includes all indexable pages, excludes author archives, tag pages, date archives
- [ ] Pagination: if used, canonical to the first page or use rel=next/prev correctly
- [ ] Tag pages: noindex (WordPress tags create duplicate/thin content)
- [ ] Author archive: noindex (local business sites don't need author archives)
- [ ] Category pages: index only if they are the Pillar pages — otherwise noindex
- [ ] Duplicate content: www vs. non-www canonical set in WordPress settings
- [ ] HTTPS: enforced site-wide with proper redirects

## Schema Markup Plan

Map schema types to page types:

| Page Type | Schema Type | Key Properties to Include |
|-----------|-------------|--------------------------|
| Homepage | LocalBusiness (most specific subtype) | name, description, address, telephone, openingHours, geo, sameAs (social profiles, GBP URL), priceRange |
| Service page | Service | name, description, provider (LocalBusiness), areaServed, offers |
| About page | AboutPage + Person (for owner) | name, description, founder, foundingDate |
| Blog/supporting | Article | headline, author, datePublished, dateModified, image |
| FAQ sections | FAQPage | question + answer pairs |
| Contact page | ContactPage + LocalBusiness | address, telephone, geo, openingHours |

**Critical for local SEO**: The LocalBusiness schema on the homepage must include:
- `@type`: Use the most specific type (e.g., "ClothingStore", "Plumber", "LegalService") — NOT generic "LocalBusiness"
- `sameAs`: Array of all social profile URLs AND the Google Business Profile URL
- `geo`: Latitude and longitude of the business
- `hasMap`: Link to Google Maps listing

## Local SEO Signal Map

Local SEO signals must be consistent and semantically aligned with the entity graph:

**NAP consistency rule**: Name, Address, Phone Number must appear identically on:
- Homepage (footer)
- Contact page
- About page
- LocalBusiness schema
- Google Business Profile
- Every citation/directory listing

**GBP alignment**:
The GBP profile must be built to match the entity map:
- GBP primary category: must match the LocalBusiness schema @type
- GBP services: must match the service entities defined in Phase 2
- GBP description: must contain the top 5 entities from Phase 2 naturally
- GBP posts: should reference supporting content from Phase 4
- GBP website URL: points to homepage
- GBP service area: matches location entities from Phase 2

## GBP Optimization Checklist

- [ ] Primary category: most specific available (check category list)
- [ ] Secondary categories: add all relevant secondary services (max 9 total)
- [ ] Services: add each service entity with its own description (150 chars each)
- [ ] Products: if applicable, add top 3 services as "products"
- [ ] Business description: 750 chars, entity-rich, includes city, service type, differentiators
- [ ] Photos: exterior (storefront), interior, team, work samples (minimum 10)
- [ ] Q&A: pre-seed 5 most common questions with answers
- [ ] Posts: publish one post per week (offer, event, or update) linking to site pages
- [ ] Reviews: set up review request workflow (request after each successful job)
- [ ] Messaging: enable GBP messaging if applicable

## WordPress Technical Implementation

**Theme**: Use a lightweight, semantic HTML theme (GeneratePress, Kadence, or Blocksy recommended)
- Reason: these themes output clean heading hierarchy and minimal render-blocking CSS

**Essential plugins**:
- Rank Math or Yoast SEO: for schema, sitemap, breadcrumbs, meta tags
- WP Rocket or LiteSpeed Cache: for page speed and Core Web Vitals
- ShortPixel or Imagify: for image compression/WebP conversion
- Internal Link Juicer (optional): automate internal link insertion based on keywords

**Permalink structure**: `/post-name/` (Settings → Permalinks → Post name)

**Page speed targets (Core Web Vitals)**:
- LCP < 2.5s
- CLS < 0.1
- INP < 200ms
- Mobile score > 80 on PageSpeed Insights

**WordPress-specific entity SEO**:
- Set site tagline (Settings → General → Tagline) to include primary entity + city
- Use Rank Math's Knowledge Graph settings to connect the site to the business entity
- Add Organization or LocalBusiness JSON-LD via Rank Math, not manually in theme code

## Output format

Write `05-technical-local.md` with H2 sections: Site Architecture, URL Structure Rules, Internal Link Flow Rules, Crawl Efficiency Checklist, Schema Markup Plan, Local SEO Signal Map, GBP Checklist, WordPress Implementation.
