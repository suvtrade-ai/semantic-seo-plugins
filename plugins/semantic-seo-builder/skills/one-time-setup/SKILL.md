---
name: one-time-setup
description: Complete one-time site launch checklist for a Semantic SEO local business website. Covers Hostinger account setup, domain selection, WordPress installation, theme and plugin configuration, Rank Math SEO setup, Google Business Profile creation and optimization, GA4 and Google Search Console connection. Produces a setup-checklist.md the user can track progress in. Use when the user says "one-time setup", "/one-time-setup", "launch checklist", "how do I set up my site", "first time setup", "I just bought hosting", "set up WordPress", "set up GBP", "connect analytics", "site launch steps", or "what do I do first".
tools:
  - Write
  - Read
---

# One-Time Site Launch Setup

This skill produces a `setup-checklist.md` file the user can track progress against. Every task is a checkbox. Run through each section in order.

Read `05-technical-local.md` if it exists for business-specific details (URL structure, schema types, GBP categories). Read `04-content-system.md` for page list. If neither exists, ask for: business name, domain, hosting platform, WordPress admin URL.

Produce `setup-checklist.md` with the following sections. For each checkbox, write a one-line instruction that a non-technical person can follow.

---

## Section 1 — Domain & Hosting (Hostinger)

```
### Domain & Hosting
- [ ] Sign up at hostinger.com — choose WordPress Hosting (Business or Premium plan)
- [ ] Register your domain during checkout (tip: use [businessname][city].com or .co.uk)
- [ ] In hPanel → Hosting → Manage → Auto Installer → WordPress → complete installation
       - Site name: [business name]
       - Admin username: NOT "admin" — choose something unique
       - Admin password: 12+ characters — save it in a password manager
       - Admin email: your main business email
- [ ] Log in to WordPress at: yourdomain.com/wp-admin
- [ ] Enable free SSL: hPanel → Hosting → Manage → SSL → Force HTTPS → Enable
```

## Section 2 — WordPress Core Settings

```
### WordPress Core Settings
- [ ] Settings → General → Site Title: [business name]
- [ ] Settings → General → Tagline: [primary service] in [city]
- [ ] Settings → General → Timezone: set to your local timezone
- [ ] Settings → Permalinks → Post name → Save Changes
- [ ] Settings → Discussion → uncheck "Allow anyone to post comments" (prevents spam)
- [ ] Users → Your Profile → update display name and email
- [ ] Delete the default "Hello World" post: Posts → Trash it
- [ ] Delete the default "Sample Page": Pages → Trash it
```

## Section 3 — Theme Installation

```
### Theme
- [ ] Appearance → Themes → Add New → search "GeneratePress" → Install → Activate
- [ ] Appearance → Customize → Site Identity → upload your business logo
- [ ] Appearance → Customize → Colors → set your brand primary color
- [ ] Appearance → Customize → Typography → set heading font (Lato or Inter recommended)
- [ ] Appearance → Widgets → Footer → add Text widget with full NAP:
       [Business Name]
       [Full Street Address]
       [City, Postcode/ZIP]
       Phone: [phone number]
```

## Section 4 — Essential Plugins

```
### Plugins
- [ ] Plugins → Add New → install and activate "Rank Math SEO"
- [ ] Plugins → Add New → install and activate "LiteSpeed Cache" (if on Hostinger) OR "WP Rocket"
- [ ] Plugins → Add New → install and activate "ShortPixel Image Optimizer"
       → Get free API key at shortpixel.com → paste in plugin settings
- [ ] Plugins → Add New → install and activate "WPForms Lite" (contact form)
- [ ] Plugins → Add New → install and activate "UpdraftPlus" (backups)
       → UpdraftPlus → Settings → schedule automatic backups weekly → connect Google Drive
- [ ] Delete unused default plugins: Akismet (unless needed), Hello Dolly
```

## Section 5 — Rank Math SEO Configuration

```
### Rank Math SEO Setup
- [ ] Rank Math → Setup Wizard → Start Wizard → choose Advanced Mode
- [ ] Connect Google account when prompted (links to Search Console automatically)
- [ ] Website Type: Local Business
- [ ] Business Name: [exact business name — must match GBP exactly]
- [ ] Business Type: [most specific type, e.g., Plumber / ClothingStore / LegalService]
- [ ] Address: full street address
- [ ] Phone: [phone number — must match GBP exactly]
- [ ] Business Hours: fill in all days
- [ ] Enable: Breadcrumbs, Rich Snippets, Local SEO
- [ ] Finish wizard

- [ ] Rank Math → Titles & Meta → Homepage → Meta Title: [Business Name] — [Service] in [City]
- [ ] Rank Math → Titles & Meta → Homepage → Meta Description: [150 chars describing service + city + key benefit]
- [ ] Rank Math → Local SEO → enable Local SEO module → fill in:
       - Google Maps URL: paste your GBP map link
       - All social profile URLs
- [ ] Rank Math → Sitemap → enable XML sitemap → note the sitemap URL (yourdomain.com/sitemap_index.xml)
- [ ] Rank Math → Analytics → paste GA4 Measurement ID (from Section 7 below)
```

## Section 6 — Build & Publish Pages

```
### Site Pages (build in this order)
- [ ] Homepage (/) — write content based on 04-content-system.md homepage brief
       → Add LocalBusiness schema via Rank Math → Schema → LocalBusiness
       → Publish
- [ ] [Service Page 1] — write from content brief → add Service schema → Publish
- [ ] [Service Page 2] — write from content brief → add Service schema → Publish
- [ ] [Service Page 3] — write from content brief → add Service schema → Publish
- [ ] About Page → write about the business and owner → add AboutPage schema → Publish
- [ ] Contact Page → add WPForms contact form → add NAP → add Google Map embed → Publish
- [ ] Set homepage: Settings → Reading → "A static page" → select your Homepage page

- [ ] Set up Navigation menu: Appearance → Menus → create menu with:
       Homepage, [Service 1], [Service 2], [About], [Contact]
       → Assign to "Primary Menu"

- [ ] Test contact form by submitting it yourself
- [ ] Check every page on mobile (use your phone — open each URL)
```

## Section 7 — Google Analytics 4 (GA4)

```
### GA4 Setup
- [ ] Go to analytics.google.com → sign in → Admin → Create Account
- [ ] Account name: [business name] → Property name: [business name] Website
- [ ] Country, currency, timezone: fill in accurately
- [ ] Platform: Web → enter your domain → Create stream
- [ ] Copy the Measurement ID (starts with G-)
- [ ] In WordPress: Rank Math → General Settings → Analytics → paste Measurement ID → Save
- [ ] Wait 24 hours, then return to GA4 → Realtime → visit your own site to confirm tracking
```

## Section 8 — Google Search Console (GSC)

```
### Google Search Console
- [ ] Go to search.google.com/search-console → Add Property → Domain → enter yourdomain.com
- [ ] Verification: choose "HTML tag" method
       → Rank Math → General Settings → Webmaster Tools → paste the GSC verification code → Save
       → Back in GSC → Verify
- [ ] GSC → Sitemaps → Add sitemap → paste: yourdomain.com/sitemap_index.xml → Submit
- [ ] GSC → Settings → confirm property ownership and email
```

## Section 9 — Google Business Profile (GBP)

```
### Google Business Profile
- [ ] Go to google.com/business → Manage now → sign in
- [ ] Search your business name → claim if found, or "Add your business to Google"
- [ ] Business name: [EXACT name as on your website footer]
- [ ] Primary category: [from 05-technical-local.md GBP section]
- [ ] Add secondary categories (up to 9 total)
- [ ] Physical location: Yes/No based on your business type
- [ ] Address: [EXACT address as on website]
- [ ] Phone: [EXACT phone as on website]
- [ ] Website: yourdomain.com (homepage)
- [ ] Business hours: fill in all days including holidays
- [ ] Complete verification (postcard / phone / video — follow Google's instructions)

After verification:
- [ ] Services: add each service from 05-technical-local.md → add individual description (150 chars each)
- [ ] Description: write 750 characters using the description from Phase 5 output
- [ ] Photos: upload minimum 10 photos (exterior, interior, team, work samples)
- [ ] Q&A: add 5 common questions and answer them yourself (prevents incorrect public answers)
- [ ] GBP website link: confirm it points to your homepage

Verify NAP consistency (must be IDENTICAL across all 3 locations):
- [ ] GBP name/address/phone matches website footer exactly
- [ ] GBP name/address/phone matches Rank Math Local SEO settings exactly
- [ ] GBP name/address/phone matches LocalBusiness schema on homepage exactly

**Zernio GBP integration (for ongoing automated management):**
- [ ] Connect your GBP to Zernio (ask your Cowork setup for the Zernio connection link)
- [ ] Once connected, weekly GBP posts will be published automatically by the `/weekly-pipeline` skill
- [ ] Monthly review responses and Q&A monitoring will run automatically via `/monthly-audit`
```

## Section 10 — Page Speed & Final Checks

```
### Speed & Launch
- [ ] LiteSpeed Cache → Settings → Cache → enable Page Cache, Browser Cache, CSS/JS Minify
- [ ] ShortPixel → Bulk Optimize all uploaded images → check WebP is enabled
- [ ] Check PageSpeed: go to pagespeed.web.dev → enter your URL → target: Mobile score > 70
- [ ] Check mobile rendering: open each main page on your phone
- [ ] Check SSL: yourdomain.com should show padlock (https) in browser bar
- [ ] Submit sitemap to Bing Webmaster Tools: bing.com/webmasters → add site → submit sitemap
- [ ] robots.txt: confirm it's not blocking Google → go to yourdomain.com/robots.txt
       → Should NOT contain: Disallow: /
```

---

## Step 9 — Submit site to Google (Indexing API launch)

Once all P1 pages are live and the sitemap is confirmed in Rank Math, run the indexing launch:

1. Invoke `/indexing` in launch mode
2. The indexing skill will:
   - Submit your sitemap (`yourdomain.com/sitemap.xml`) to Google Search Console
   - Batch submit all P1 and P2 pages to the Google Indexing API for priority crawling
   - Create `indexing-log.md` to track submission status
3. Wait 30–60 minutes, then run the crawl check to confirm pages are queued for indexing

**Requires:** `gsc-service-account.json` in the project root (see `/indexing` for setup instructions).

If the service account is not yet configured, submit the sitemap manually:
- GSC → Sitemaps → Add new sitemap → `sitemap.xml` → Submit
- Then use GSC → URL Inspection → Request Indexing for each P1 page individually

**Pages to submit at launch (in priority order):**
1. Homepage
2. All P1 service pages (bespoke suits, alterations, tailoring city page, etc.)
3. All P2 subtopic pages
4. Contact page

Do NOT submit supporting blog articles at launch — these will be submitted individually via `/weekly-pipeline` Step 4c as they are published.

---

## Launch checklist summary

After completing all steps, confirm:
- [ ] Domain live and SSL active
- [ ] WordPress installed with Rank Math configured
- [ ] All P1 pages published and set to index
- [ ] Sitemap submitted to GSC
- [ ] P1/P2 pages submitted to Indexing API
- [ ] GA4 tracking confirmed (pageview firing on homepage)
- [ ] GBP created and verified
- [ ] `indexing-log.md` created with launch submissions logged

---

## Step 10 — Schema validation

After publishing all P1 pages, validate that schema is rendering correctly.

**For each P1 page:**
1. Go to [Google Rich Results Test](https://search.google.com/test/rich-results) → paste the live URL
2. Confirm the correct schema type is detected (LocalBusiness, Service, FAQPage)
3. Check for errors — missing `name`, `address`, `telephone`, or `url` fields break eligibility

**Common errors to fix in Rank Math:**
- `telephone` missing → add under Schema → LocalBusiness → telephone
- `address` incomplete → PostalCode and AddressCountry required
- FAQ schema not detected → ensure FAQ block is added via Rank Math Schema, not just HTML
- Multiple conflicting schema types on one page → remove duplicates

**Add to setup checklist:**
- [ ] Homepage LocalBusiness schema passes Rich Results Test
- [ ] Each P1 service page has Service schema with no errors
- [ ] FAQPage schema validated on pages with FAQ sections

## Schema — floorLevel Check

If the business is inside a multi-story building, verify the homepage LocalBusiness JSON-LD includes:
- `"floorLevel": "[floor value]"` placed OUTSIDE the `PostalAddress` wrapper
- Value from `01-foundation.md` Phase 1 intake

Missing `floorLevel` on a mall/tower business = critical schema gap. Add via Rank Math → Custom Schema before launch.

---

## Step 11 — Technical SEO baseline (run after all P1 pages are live)

Run these four checks in sequence after all P1 pages are published and before calling the launch done. Issues found here are easier to fix before Google indexes the site.

### 11a — Technical SEO audit with seo-technical

```
/seo technical [domain]
```

`seo-technical` checks 9 categories: crawlability, indexability, security, URL structure, mobile, Core Web Vitals, structured data, JavaScript rendering, and IndexNow protocol.

**Critical checks at launch:**
- robots.txt must NOT block `/` or any service page slug
- No pages should have `noindex` set in Rank Math (common mistake left from draft mode)
- HTTPS redirect confirmed: HTTP → HTTPS for all URLs
- Mobile rendering passes: GeneratePress is mobile-friendly but confirm on the actual domain
- All canonical tags are self-referencing (`/bespoke-suit-tailor/` points to itself — not to homepage)

Fix any CRITICAL or HIGH severity findings before submitting pages to Google Indexing API.

### 11b — Sitemap validation with seo-sitemap

Before submitting the sitemap to GSC in Section 8, validate it:

```
/seo sitemap [domain]
```

`seo-sitemap` checks:
- All P1 service pages appear in the sitemap
- No noindexed pages are included (they must not be in the sitemap)
- `lastmod` dates are accurate
- URL count matches the number of published pages

**Common issue at launch:** Rank Math sometimes excludes the homepage from the sitemap when it's set as a "static front page" — confirm `yourdomain.com/` appears in the sitemap XML before submitting.

After validation passes, submit: `yourdomain.com/sitemap_index.xml` to GSC → Sitemaps.

### 11c — Image SEO audit with seo-images

After ShortPixel has processed and converted all uploaded images:

```
/seo images [domain]
```

`seo-images` checks:
- Alt text present on ALL images (missing alt = accessibility failure + lost image search traffic)
- Alt text is descriptive and includes the primary service entity and location (e.g., "bespoke suit tailoring in Ko Samui" — NOT "image" or the raw filename)
- All images served as WebP format
- No image file over 100KB after compression (oversized images trigger LCP failures)
- Hero/featured images have an `ImageObject` schema entry

**Target standard:** Every image on every P1 page has unique, entity-specific alt text. Re-upload any image ShortPixel failed to convert.

### 11d — SEO drift baseline (one-time, run at launch only)

After all P1 pages are live, all technical checks pass, and the first batch of indexing requests has been submitted — capture the initial baseline snapshot:

```
/seo drift [domain] --baseline
```

This creates the reference snapshot for all future weekly drift checks. `seo-drift` records 13 SEO-critical elements per tracked page:
- Title tag, meta description, canonical URL
- robots meta tag, H1, H2/H3 structure  
- JSON-LD schema hash, Open Graph tags
- Core Web Vitals baseline, HTTP status, full HTML hash

**Run this ONCE after launch when the site is in its correct, validated state.** Running on a broken or incomplete site records the wrong baseline and makes future drift reports meaningless.

After the baseline is captured, the weekly pipeline's Step W9 (`/seo drift --check`) compares against it automatically. No need to re-run `--baseline` unless you intentionally redesign the site.

**Add to setup-checklist.md:**

```
### Technical SEO Baseline (Step 11)
- [ ] /seo technical [domain] — no critical issues found
- [ ] /seo sitemap [domain] — all P1 pages included, no noindexed pages in sitemap
- [ ] /seo images [domain] — all images have descriptive alt text, WebP format confirmed
- [ ] /seo drift [domain] --baseline — initial snapshot captured (run ONCE only)
```
