---
name: html-pages
description: Phase 6 of the Semantic SEO pipeline. Generates fully responsive, mobile-optimised static HTML preview pages from the Phase 1-5 blueprint documents. Reads 01-foundation.md through 05-technical-local.md, builds every page in the content inventory (homepage, service pages, about, contact, blog index, individual blog articles), generates images via Kie API for each section, and saves all files to a preview/ directory. Use when the user says "generate pages", "build HTML pages", "create preview pages", "html-pages", "/html-pages", or when workflow reaches Phase 6.
tools:
  - Read
  - Write
  - Bash
---

# Phase 6 — Static HTML Page Builder

Reads the complete Semantic SEO blueprint from Phases 1–5 and builds every page as a responsive, mobile-optimised static HTML file with real images. Output goes to `preview/`.

---

## Step 1 — Read the blueprint

Read all phase documents before writing a single line of HTML:

```
01-foundation.md       → business name, NAP, services, ICPs, tone
02-semantic-research.md → primary entities, attributes — use in copy and alt text
03-topical-map.md      → page hierarchy, internal link targets
04-content-system.md   → content inventory table, semantic briefs, written copy
05-technical-local.md  → URL structure, schema types per page, GBP alignment
```

Extract and store:
- Business name, address, phone, city, country
- Primary service entities (from semantic research)
- Page list from content inventory table
- URL slug for each page
- Schema type for each page (LocalBusiness, Service, BlogPosting, etc.)
- Brand tone (from foundation ICP section)

---

## Step 2 — Plan the page set

Build a page manifest from the content inventory. Priority order:

```
P1 — Must build first:
  index.html            Homepage
  services/             One file per pillar service
  about.html            About / story
  contact.html          Contact + map embed

P2 — Build after P1:
  blog/index.html       Blog listing page
  blog/[slug].html      Each article from weekly-pipeline

P3 — Build after P2:
  [city].html           Location pages (if multi-location)
  faq.html              FAQ page
```

Show the user the page manifest and confirm before building:

```
Ready to build [N] pages for [Business Name]:

  P1 (4 pages):
    index.html — Homepage
    services/bespoke-suits.html
    services/shirt-tailoring.html
    about.html
    contact.html

  P2 (will build after P1):
    blog/index.html
    blog/[article-slug].html

Shall I proceed with P1 first?
```

---

## Step 3 — Global design system

Every page shares one CSS file (`preview/css/style.css`) and one JS file (`preview/js/main.js`). Never inline full styles — reference the shared files. Page-specific overrides go in a `<style>` block in `<head>`.

### CSS variables (dark professional theme)
```css
:root {
  --bg:      #0f0f0f;
  --bg2:     #1a1a1a;
  --bg3:     #222222;
  --border:  #2a2a2a;
  --text:    #e8e8e8;
  --muted:   #888888;
  --accent:  #c9a96e;   /* gold — replace with brand colour from foundation */
  --accent2: #a07840;
  --radius:  4px;
  --max:     1100px;
  --gap:     clamp(16px, 4vw, 40px);
}
```

### Base reset and mobile-first typography
```css
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

html { font-size: 16px; scroll-behavior: smooth; }

body {
  background: var(--bg);
  color: var(--text);
  font-family: 'Inter', system-ui, sans-serif;
  line-height: 1.7;
  font-size: clamp(0.95rem, 2vw, 1.05rem);
}

h1 { font-size: clamp(1.8rem, 5vw, 3rem); line-height: 1.2; }
h2 { font-size: clamp(1.4rem, 3.5vw, 2.1rem); line-height: 1.3; }
h3 { font-size: clamp(1.1rem, 2.5vw, 1.4rem); }

p { max-width: 68ch; }

a { color: var(--accent); text-decoration: none; }
a:hover { text-decoration: underline; }

img { max-width: 100%; height: auto; display: block; }

.container {
  width: 100%;
  max-width: var(--max);
  margin: 0 auto;
  padding: 0 var(--gap);
}

section { padding: clamp(40px, 8vw, 80px) 0; }
```

### Navigation (hamburger on mobile)
```html
<nav class="nav" role="navigation" aria-label="Main navigation">
  <div class="container nav__inner">
    <a href="/" class="nav__logo" aria-label="[Business Name] home">[Business Name]</a>
    <button class="nav__toggle" aria-label="Open menu" aria-expanded="false" aria-controls="nav-menu">
      <span></span><span></span><span></span>
    </button>
    <ul class="nav__menu" id="nav-menu" role="list">
      <li><a href="/">Home</a></li>
      <li><a href="/services/">Services</a></li>
      <li><a href="/about.html">About</a></li>
      <li><a href="/blog/">Blog</a></li>
      <li><a href="/contact.html" class="nav__cta">Book a Fitting</a></li>
    </ul>
  </div>
</nav>
```

```css
.nav {
  position: sticky;
  top: 0;
  z-index: 100;
  background: var(--bg);
  border-bottom: 1px solid var(--border);
}
.nav__inner {
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 60px;
}
.nav__logo {
  font-size: 1.1rem;
  font-weight: 600;
  letter-spacing: 0.04em;
  color: var(--text);
  text-decoration: none;
}
.nav__menu {
  display: flex;
  gap: 28px;
  list-style: none;
  align-items: center;
}
.nav__cta {
  background: var(--accent);
  color: #000;
  padding: 8px 18px;
  border-radius: var(--radius);
  font-weight: 600;
  font-size: 0.9rem;
}
.nav__toggle {
  display: none;
  flex-direction: column;
  gap: 5px;
  background: none;
  border: none;
  cursor: pointer;
  padding: 6px;
}
.nav__toggle span {
  display: block;
  width: 24px;
  height: 2px;
  background: var(--text);
  transition: transform 0.2s, opacity 0.2s;
}

@media (max-width: 720px) {
  .nav__toggle { display: flex; }
  .nav__menu {
    display: none;
    flex-direction: column;
    gap: 0;
    position: absolute;
    top: 60px;
    left: 0;
    right: 0;
    background: var(--bg2);
    border-bottom: 1px solid var(--border);
    padding: 16px 0;
  }
  .nav__menu.is-open { display: flex; }
  .nav__menu li { width: 100%; }
  .nav__menu a { display: block; padding: 12px 24px; }
  .nav__cta { margin: 8px 24px; border-radius: var(--radius); }
}
```

```js
// main.js — hamburger toggle
const toggle = document.querySelector('.nav__toggle');
const menu = document.querySelector('.nav__menu');
if (toggle && menu) {
  toggle.addEventListener('click', () => {
    const open = menu.classList.toggle('is-open');
    toggle.setAttribute('aria-expanded', open);
    toggle.setAttribute('aria-label', open ? 'Close menu' : 'Open menu');
  });
}
```

### Footer with full NAP
```html
<footer class="footer">
  <div class="container footer__inner">
    <div class="footer__brand">
      <strong>[Business Name]</strong>
      <p>[Tagline — primary service in city]</p>
    </div>
    <div class="footer__nav">
      <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/services/">Services</a></li>
        <li><a href="/about.html">About</a></li>
        <li><a href="/blog/">Blog</a></li>
        <li><a href="/contact.html">Contact</a></li>
      </ul>
    </div>
    <div class="footer__contact">
      <p>[Full Street Address]</p>
      <p>[City, Postcode/Area]</p>
      <p><a href="tel:[phone]">[phone]</a></p>
      <p><a href="mailto:[email]">[email]</a></p>
    </div>
  </div>
  <div class="footer__bottom">
    <div class="container">
      <p>&copy; [Year] [Business Name]. All rights reserved.</p>
    </div>
  </div>
</footer>
```

```css
.footer {
  background: var(--bg2);
  border-top: 1px solid var(--border);
  padding: 48px 0 0;
  margin-top: 80px;
}
.footer__inner {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 32px;
  padding-bottom: 40px;
}
.footer__bottom {
  border-top: 1px solid var(--border);
  padding: 16px 0;
  font-size: 0.82rem;
  color: var(--muted);
}
```

---

## Step 4 — Page templates

### Page build loop — order for EVERY page

Build each page in this exact order. Do not skip any step:

```
For each page in the manifest:
  1. Write full HTML body content (sections, nav, footer)
  2. Generate schema JSON-LD for this page type → inject into <head>
  3. Run mobile checklist — fix any failures before continuing
  4. Confirm image list with user → generate images via Kie API → insert into HTML
  5. Save file to preview/
  6. Move to next page
```

Schema is always Step 2 — generated immediately after the HTML body is written, before images. Every page gets JSON-LD in `<head>` before the file is saved. Never save a page without its schema.

### HTML document shell (every page)
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="description" content="[meta description from content brief — 140-160 chars]">
  <title>[Page Title] | [Business Name]</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap">
  <link rel="stylesheet" href="/css/style.css">

  <!-- SCHEMA — generated per page type, see Step 4a below -->
  <script type="application/ld+json">
  { ... }
  </script>

  <style>
    /* page-specific overrides only */
  </style>
</head>
<body>
  [nav]
  <main>
    [page content]
  </main>
  [footer]
  <script src="/js/main.js" defer></script>
</body>
</html>
```

---

## Step 4a — Schema JSON-LD per page (runs immediately after body is written)

Read `05-technical-local.md` for the schema type assigned to each page. Generate the full JSON-LD and insert it into `<head>` before saving. Use real business data — no placeholders in the final output.

### Homepage — LocalBusiness (or most specific subtype)
```json
{
  "@context": "https://schema.org",
  "@type": "[BusinessType from 05-technical-local.md — e.g. ClothingStore, LocalBusiness]",
  "name": "[Business Name]",
  "description": "[Primary service — 1 sentence, entity-rich]",
  "url": "https://[domain]/",
  "telephone": "[phone]",
  "email": "[email if available]",
  "address": {
    "@type": "PostalAddress",
    "streetAddress": "[street address]",
    "addressLocality": "[city]",
    "addressRegion": "[state/province if applicable]",
    "postalCode": "[postcode]",
    "addressCountry": "[2-letter country code]"
  },
  "floorLevel": "[floor — e.g. 2nd Floor, only if business is in multi-story building]",
  "geo": {
    "@type": "GeoCoordinates",
    "latitude": "[lat from 05-technical-local.md if available]",
    "longitude": "[lng from 05-technical-local.md if available]"
  },
  "openingHoursSpecification": [
    {
      "@type": "OpeningHoursSpecification",
      "dayOfWeek": ["Monday","Tuesday","Wednesday","Thursday","Friday","Saturday"],
      "opens": "10:00",
      "closes": "20:00"
    }
  ],
  "image": "images/[hero-image-filename].webp",
  "priceRange": "$$",
  "currenciesAccepted": "[currency — e.g. THB, USD, GBP]",
  "paymentAccepted": "Cash, Credit Card",
  "hasMap": "https://maps.google.com/?q=[encoded address]",
  "sameAs": [
    "https://www.facebook.com/[handle]",
    "https://www.instagram.com/[handle]"
  ]
}
```

**Rule:** `floorLevel` sits directly on the LocalBusiness object — never inside `address`. Only include it if the business is in a multi-story building (from Phase 1 intake).

### Service pages — Service + LocalBusiness provider
```json
{
  "@context": "https://schema.org",
  "@type": "Service",
  "name": "[Service Name — e.g. Bespoke Suit Tailoring]",
  "description": "[Service description — entity-rich, 1-2 sentences]",
  "provider": {
    "@type": "[BusinessType]",
    "name": "[Business Name]",
    "url": "https://[domain]/",
    "telephone": "[phone]",
    "address": {
      "@type": "PostalAddress",
      "streetAddress": "[street]",
      "addressLocality": "[city]",
      "addressCountry": "[country code]"
    }
  },
  "areaServed": {
    "@type": "City",
    "name": "[City]"
  },
  "serviceType": "[service category — e.g. Bespoke Tailoring]",
  "url": "https://[domain]/services/[slug]/"
}
```

**Also add FAQPage schema on service pages** (from the FAQ section at the bottom of the page):
```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "[Q1 from FAQ section]",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "[A1 — full answer text]"
      }
    },
    {
      "@type": "Question",
      "name": "[Q2]",
      "acceptedAnswer": { "@type": "Answer", "text": "[A2]" }
    }
  ]
}
```

Include both schemas in `<head>` as two separate `<script type="application/ld+json">` blocks.

### About page — AboutPage + LocalBusiness
```json
{
  "@context": "https://schema.org",
  "@type": "AboutPage",
  "name": "About [Business Name]",
  "url": "https://[domain]/about.html",
  "description": "[About page summary]",
  "publisher": {
    "@type": "[BusinessType]",
    "name": "[Business Name]",
    "url": "https://[domain]/"
  }
}
```

### Contact page — ContactPage + LocalBusiness
```json
{
  "@context": "https://schema.org",
  "@type": "ContactPage",
  "name": "Contact [Business Name]",
  "url": "https://[domain]/contact.html",
  "description": "Contact [Business Name] for appointments, fittings, and enquiries.",
  "publisher": {
    "@type": "[BusinessType]",
    "name": "[Business Name]",
    "telephone": "[phone]",
    "address": {
      "@type": "PostalAddress",
      "streetAddress": "[street]",
      "addressLocality": "[city]",
      "addressCountry": "[country code]"
    }
  }
}
```

### Blog index — CollectionPage
```json
{
  "@context": "https://schema.org",
  "@type": "CollectionPage",
  "name": "[Business Name] Blog",
  "url": "https://[domain]/blog/",
  "description": "[Niche] tips, guides, and advice from [Business Name] in [City].",
  "publisher": {
    "@type": "[BusinessType]",
    "name": "[Business Name]",
    "url": "https://[domain]/"
  }
}
```

### Blog article — BlogPosting + FAQPage + BreadcrumbList
```json
{
  "@context": "https://schema.org",
  "@type": "BlogPosting",
  "headline": "[Article H1]",
  "description": "[Meta description]",
  "url": "https://[domain]/blog/[slug].html",
  "datePublished": "[YYYY-MM-DD]",
  "dateModified": "[YYYY-MM-DD]",
  "author": {
    "@type": "Organization",
    "name": "[Business Name]",
    "url": "https://[domain]/"
  },
  "publisher": {
    "@type": "Organization",
    "name": "[Business Name]",
    "url": "https://[domain]/"
  },
  "image": "images/[article-hero].webp",
  "mainEntityOfPage": {
    "@type": "WebPage",
    "@id": "https://[domain]/blog/[slug].html"
  }
}
```

Add BreadcrumbList on every page:
```json
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    { "@type": "ListItem", "position": 1, "name": "Home", "item": "https://[domain]/" },
    { "@type": "ListItem", "position": 2, "name": "[Page Name]", "item": "https://[domain]/[slug]" }
  ]
}
```

### Schema validation checklist (after generating each page's schema)
```
[ ] All @type values match what's in 05-technical-local.md
[ ] floorLevel present on LocalBusiness if multi-story building (from Phase 1 intake)
[ ] floorLevel NOT inside address block
[ ] No placeholder text remaining — all fields filled with real business data
[ ] FAQPage included on all service pages (from FAQ section)
[ ] BreadcrumbList included on every page
[ ] Two separate <script> blocks on service pages (Service + FAQPage)
[ ] telephone matches NAP in footer exactly
[ ] address matches NAP in footer exactly
```

### Homepage sections (in order)
1. Hero — headline (H1 with primary keyword), subheadline, two CTAs (primary + secondary), hero image
2. Services overview — H2, 3–4 service cards with icon/image, link to service page
3. Why us / differentiators — H2, 3 columns of key strengths from Phase 1 ICP research
