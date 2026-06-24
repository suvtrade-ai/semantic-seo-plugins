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
  <script type="application/ld+json">
  [schema JSON-LD — type from 05-technical-local.md]
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

### Homepage sections (in order)
1. Hero — headline (H1 with primary keyword), subheadline, two CTAs (primary + secondary), hero image
2. Services overview — H2, 3–4 service cards with icon/image, link to service page
3. Why us / differentiators — H2, 3 columns of key strengths from Phase 1 ICP research
4. Process — H2, numbered steps, hands/process image
5. Fabric / materials (if relevant) — H2, flat lay image, brief copy
6. Testimonials — H2, 2–3 quote cards (pull from foundation research or write realistic placeholders)
7. Location / find us — H2, address, click-to-call button, Google Maps embed
8. Blog preview — H2, latest 2–3 articles as cards (link to blog/)
9. Final CTA banner — strong action headline, single button

### Service page sections
1. Hero — H1 with service keyword, image of the specific garment/service
2. What we offer — H2, detailed description using entities from semantic research
3. Our process for this service — H2, step-by-step
4. Fabric/material options (if relevant) — H2, flat lay image
5. Pricing guidance (optional) — H2, "starting from" ranges or "custom quote"
6. Related services — H2, 2–3 cards linking to sibling service pages
7. FAQ for this service — H2, 4–6 Q&A using PAA questions from topical map
8. CTA — "Book a fitting" button, phone number

### About page sections
1. Hero — H1, interior workshop image
2. Our story — H2, founder narrative (draw from foundation brand voice)
3. Craft & expertise — H2, skills, years of experience, specialisms
4. Workshop / atelier — H2, interior image, location description
5. Values — H2, 3 core values
6. CTA — "Meet us" or "Book an appointment"

### Contact page sections
1. Header — H1, brief welcoming line
2. Contact details — phone (click-to-call `<a href="tel:...">`), email, address, hours
3. Google Maps embed — responsive iframe
4. Contact form — name, email, phone, message, service dropdown, submit
5. Directions — brief landmark text from address in 05-technical-local.md

### Blog index sections
1. Header — H1 "[Niche] Blog — Tips, Guides & Inspiration"
2. Featured article — latest article, large card with image
3. Article grid — remaining articles in 2–3 column responsive grid
4. Categories sidebar (on desktop) — service-aligned categories

---

## Step 5 — Image generation per page

After writing each page, immediately call `/image-gen` for that page. Do not batch all pages then image all at once — do it page by page.

For each page:
1. Identify all sections that need an image (Step 1 of image-gen skill)
2. Show the confirmation list: "Ready to generate [N] images for [page]. Proceed?"
3. After approval, fire all image jobs for that page simultaneously
4. Download, compress to <100KB, insert into page HTML
5. Move to the next page

Images must be unique per section and per page. Track all generated filenames in a running list for the entire session. No filename may be reused.

---

## Step 6 — Mobile optimisation checklist

Before saving each page file, verify:

```
Mobile checks:
[ ] Viewport meta tag present: <meta name="viewport" content="width=device-width, initial-scale=1.0">
[ ] No horizontal scroll — no element wider than 100vw
[ ] Touch targets minimum 44x44px (buttons, nav links)
[ ] Font sizes use clamp() — never fixed px below 14px
[ ] Images use clamp() heights — no fixed pixel height on mobile
[ ] Hero image height: clamp(200px, 50vw, 500px) on mobile
[ ] Nav collapses to hamburger below 720px
[ ] Grid columns collapse to 1 column below 640px
[ ] Click-to-call on all phone numbers: <a href="tel:[phone]">
[ ] Google Maps embed has max-width: 100%; height: 300px on mobile
[ ] Form inputs: font-size 16px minimum (prevents iOS zoom on focus)
[ ] CTA buttons: min-width: 100% on mobile
[ ] Footer grid collapses to 1 column below 640px
```

---

## Step 7 — Schema markup per page

Pull schema types from `05-technical-local.md`. Every page gets JSON-LD in `<head>`.

### Homepage — LocalBusiness
```json
{
  "@context": "https://schema.org",
  "@type": "[BusinessType from 05-technical-local.md]",
  "name": "[Business Name]",
  "description": "[Primary service description]",
  "url": "https://[domain]/",
  "telephone": "[phone]",
  "address": {
    "@type": "PostalAddress",
    "streetAddress": "[street]",
    "addressLocality": "[city]",
    "addressCountry": "[country code]"
  },
  "floorLevel": "[floor — e.g. 2nd Floor]",
  "openingHoursSpecification": [...],
  "image": "images/[hero-image].webp",
  "priceRange": "$$"
}
```

Note: `floorLevel` sits outside `address`, directly on the LocalBusiness object.

### Service pages — Service schema
```json
{
  "@context": "https://schema.org",
  "@type": "Service",
  "name": "[Service Name]",
  "provider": {
    "@type": "[BusinessType]",
    "name": "[Business Name]",
    "url": "https://[domain]/"
  },
  "areaServed": "[City]",
  "description": "[Service description]"
}
```

### Blog articles — BlogPosting + FAQPage
Pull from `blog-schema` output if available. Otherwise generate inline.

---

## Step 8 — Internal linking

Every page must link to at least 3 other pages using anchor text drawn from the internal link map in `04-content-system.md`.

- Service pages link to each other ("See also: [related service]")
- Homepage links to all P1 service pages
- Blog articles link to the relevant service page
- All pages link to contact.html in the nav and in at least one in-body CTA

---

## Step 9 — Save files

```
preview/
  index.html
  about.html
  contact.html
  css/
    style.css
  js/
    main.js
  images/
    [all webp files]
  services/
    [service-slug].html
    (one file per pillar service)
  blog/
    index.html
    [article-slug].html
```

Name each HTML file using the URL slug from `05-technical-local.md`.

---

## Step 10 — Completion report

After all pages are built:

```
Phase 6 complete — Static HTML pages built

Pages created:
  preview/index.html                     — Homepage
  preview/services/bespoke-suits.html    — Bespoke Suits
  preview/services/shirt-tailoring.html  — Shirt Tailoring
  preview/about.html                     — About
  preview/contact.html                   — Contact
  preview/blog/index.html                — Blog Index

Images:
  [N] images generated, all under 100KB
  Total image payload: [X]KB

Mobile: all pages pass mobile checklist
Schema: LocalBusiness on homepage, Service on service pages, FAQPage on service pages

Next: Phase 7 — Site Launch (/one-time-setup)
```

---

## Rules

- No emojis anywhere in HTML output — not in headings, buttons, copy, or comments
- No inline styles except where CSS variables are used in a `style` attribute
- All phone numbers must be click-to-call: `<a href="tel:[phone]">`
- All images must have descriptive, entity-based alt text (never empty, never "image")
- All external links (Google Maps embed, fonts) use `rel="noopener"` where applicable
- Copy must use real business entities from the phase documents — no generic placeholders like "Company Name" left in the final HTML
- Never use `!important` in CSS
- Grid layouts must collapse gracefully at 640px and 380px breakpoints
