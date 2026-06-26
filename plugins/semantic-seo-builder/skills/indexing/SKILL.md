---
name: indexing
description: Programmatic Google indexing via Indexing API and URL Inspection API. Three modes: launch (batch submit all pages + sitemap), weekly (submit new article + crawl check), monthly (index health audit, detect deindexed pages, re-submit). Use when user says index my site, submit to Google, crawl check, why isn't my page indexed, or /indexing.
tools:
  - Read
  - Write
  - Bash
---

# Indexing Skill — `/indexing`

Programmatic Google indexing for Semantic SEO sites. Submits pages to the Google Indexing API and checks crawl status via the URL Inspection API. Covers three modes: site launch (batch P1/P2 submission), weekly (per-article after publish), and monthly (index health audit).

---

## Trigger phrases

- `/indexing`
- "submit pages to Google"
- "index my site"
- "index new article"
- "check if my pages are indexed"
- "submit sitemap to GSC"
- "crawl check"
- "indexing setup"
- "why isn't my page indexed"

---

## Prerequisites

This skill requires Google API credentials. Ask the user to confirm which they have before proceeding.

### Service Account setup (required for Indexing API)

1. Go to [Google Cloud Console](https://console.cloud.google.com/) → New Project
2. Enable two APIs: **Google Indexing API** and **Google Search Console API**
3. Create a Service Account → download JSON key file
4. In Google Search Console → Settings → Users and permissions → Add the service account email as **Owner** (Owner access is required — Viewer is not enough)
5. Save the JSON key content as `gsc-service-account.json` in the project root

If credentials file not found, offer to walk through the 5-minute setup, then ask:
```
To use the Indexing API I need your Google Service Account JSON key.
Have you set this up in Google Cloud Console?
If yes, paste the path or share the JSON contents.
If not, I'll walk you through it now.
```

**Authentication helper — save as `get_token.py` in project root:**

```python
import json, time, jwt, requests

def get_access_token(key_file="gsc-service-account.json"):
    key = json.load(open(key_file))
    now = int(time.time())
    payload = {
        "iss": key["client_email"],
        "scope": "https://www.googleapis.com/auth/indexing https://www.googleapis.com/auth/webmasters",
        "aud": "https://oauth2.googleapis.com/token",
        "iat": now,
        "exp": now + 3600
    }
    signed = jwt.encode(payload, key["private_key"], algorithm="RS256")
    resp = requests.post("https://oauth2.googleapis.com/token", data={
        "grant_type": "urn:ietf:params:oauth2:grant-type:jwt-bearer",
        "assertion": signed
    })
    return resp.json()["access_token"]
```

Install: `pip install PyJWT cryptography requests`

Claude will generate this file during setup if it doesn't exist.

---

## Mode 1 — Launch Mode (run once when site goes live)

**Trigger:** User says "index my site", "we just launched", "submit all pages", or when `/one-time-setup` completes.

### Step 1 — Submit sitemap

Read the domain from `01-foundation.md`. Submit sitemap to GSC:

```python
SITEMAP_URL = "https://yourdomain.com/sitemap.xml"
SITE_PROPERTY = "sc-domain:yourdomain.com"

requests.put(
    f"https://www.googleapis.com/webmasters/v3/sites/{SITE_PROPERTY}/sitemaps/{SITEMAP_URL}",
    headers={"Authorization": f"Bearer {get_access_token()}"}
)
```

### Step 2 — Identify P1 and P2 pages

Read `03-topical-map.md` to extract all live P1 and P2 URLs. If files don't exist, ask user to list the live URLs.

Priority order for submission:
1. Homepage
2. All P1 service/pillar pages
3. All P2 subtopic pages
4. Contact page
5. About page

**Indexing API quota:** 200 URL submissions per day. A typical new local business site has fewer than 20 P1+P2 pages — well within quota.

### Step 3 — Batch submit all P1/P2 to Indexing API

```python
for url in priority_urls:
    response = requests.post(
        "https://indexing.googleapis.com/v3/urlNotifications:publish",
        headers={"Authorization": f"Bearer {get_access_token()}"},
        json={"url": url, "type": "URL_UPDATED"}
    )
    status = "OK" if response.status_code == 200 else f"ERROR {response.status_code}"
    print(f"{status} — {url}")
    time.sleep(1)  # avoid rate limiting
```

### Step 4 — Crawl check (run 30–60 min after submission)

```python
def inspect_url(url, site_property):
    response = requests.post(
        "https://searchconsole.googleapis.com/v1/urlInspection/index:inspect",
        headers={"Authorization": f"Bearer {get_access_token()}"},
        json={"inspectionUrl": url, "siteUrl": site_property}
    )
    result = response.json()["inspectionResult"]["indexStatusResult"]
    return {
        "verdict": result.get("verdict", "UNKNOWN"),
        "coverage": result.get("coverageState", "—"),
        "last_crawl": result.get("lastCrawlTime", "never")
    }

for url in priority_urls:
    check = inspect_url(url, SITE_PROPERTY)
    print(f"{url}")
    print(f"  Verdict: {check['verdict']} | Coverage: {check['coverage']} | Last crawl: {check['last_crawl']}")
```

**Verdict meanings:**
- `PASS` — indexed ✓
- `NEUTRAL` — crawled but not yet indexed (normal within first 48h — check again tomorrow)
- `FAIL` — blocked (check noindex tag, robots.txt, or Rank Math settings)

### Step 5 — Create indexing log

Create `indexing-log.md` in the project root:

```markdown
# Indexing Log

## Launch — [date]

### Sitemap
- Submitted: [datetime]
- URL: https://yourdomain.com/sitemap.xml

### Pages submitted
| URL | Type | Submitted | Crawl Check | Verdict |
|-----|------|-----------|-------------|---------|
| / | P1 — Homepage | [datetime] | [datetime] | PASS |
| /bespoke-suits/ | P1 | [datetime] | [datetime] | NEUTRAL |
| /alterations/ | P1 | [datetime] | [datetime] | NEUTRAL |

### Next check: [date + 48 hours]
```

---

## Mode 2 — Weekly Mode (called from /weekly-pipeline Step 4c)

**Trigger:** Automatically after WordPress publish, or manually via `/indexing weekly [url]`.

### Step 1 — Submit the published URL

```python
url = "[newly published article URL from WordPress]"

response = requests.post(
    "https://indexing.googleapis.com/v3/urlNotifications:publish",
    headers={"Authorization": f"Bearer {get_access_token()}"},
    json={"url": url, "type": "URL_UPDATED"}
)

if response.status_code == 200:
    print(f"Submitted: {url}")
else:
    print(f"Failed: {response.status_code} — {response.text}")
```

### Step 2 — Crawl check (30–60 min after submission)

Run `inspect_url(url, SITE_PROPERTY)` and report the verdict.

**If FAIL — diagnose:**
- Check page source for `<meta name="robots" content="noindex">`
- Check `robots.txt` at yourdomain.com/robots.txt — is the URL pattern blocked?
- In Rank Math on that post — is "Robots Meta" set to noindex?
- Check if the canonical tag points to a different URL

### Step 3 — Append to indexing log

```markdown
## Week of [date]
- Article: [title] → [url]
- Submitted: [datetime]
- Crawl check: [datetime]
- Verdict: [PASS / NEUTRAL / FAIL]
- Notes: [any issues]
```

If verdict is NEUTRAL within first 48h — this is normal. Log it and re-check in monthly audit.

---

## Mode 3 — Monthly Index Health Audit (called from /monthly-audit)

**Trigger:** Automatically during `/monthly-audit`, or manually via `/indexing audit`.

### Step 1 — Read indexing log

Read `indexing-log.md`. Collect all P1/P2 launch pages and all weekly articles ever submitted.

### Step 2 — Re-inspect all P1/P2 pages

Run `inspect_url` on all P1 and P2 pages and compare against previous verdicts.

Flag for re-submission:
- Any P1/P2 with current verdict `FAIL`
- Any P1/P2 with `NEUTRAL` that was submitted more than 7 days ago
- Any page where `lastCrawlTime` is older than 60 days

Flag as critical alert:
- Any page that previously showed `PASS` and now shows `NEUTRAL` or `FAIL` (deindexing signal)

### Step 3 — Re-submit flagged pages

For each flagged URL, re-submit to Indexing API and note the reason in the log.

### Step 4 — Check for deindexing causes

If any previously-indexed page is now failing:
- Was Rank Math updated? Check if robots settings changed site-wide
- Did a WordPress plugin add a noindex header?
- Did the URL change (check redirect chain)?
- Did the canonical tag change?

### Step 5 — Monthly index report

```markdown
## Index Health — [month year]

### Summary
- Total pages tracked: X
- Indexed (PASS): X / X
- Pending (NEUTRAL): X
- Blocked (FAIL): X

### P1 Status
| Page | Verdict | Last Crawled | Action Taken |
|------|---------|-------------|--------------|
| Homepage | PASS | 2 days ago | — |
| /bespoke-suits/ | PASS | 5 days ago | — |
| /alterations/ | NEUTRAL | 12 days ago | Re-submitted |

### Issues Found
[describe any problems and actions taken]

### Next audit: [date + 30 days]
```

---

## What NOT to index (noindex reference)

Apply `noindex` in Rank Math → Titles & Meta for these page types:

| Page type | Reason |
|-----------|--------|
| Thank-you / confirmation pages | Thin, no search value |
| Tag archive pages | Duplicate / thin content |
| Author archive (single-author site) | Thin content |
| Search results pages `/?s=` | Dynamic, duplicate |
| WordPress login / admin URLs | Security + irrelevant |
| Paginated archives beyond page 2 | Thin, low crawl value |
| Staging or preview URLs | Duplicate of live |
| Category pages (if thin) | Review case-by-case |

**In Rank Math:** Titles & Meta → Tags → noindex. Titles & Meta → Author → noindex.

Category pages: if a category has fewer than 5 posts, set to noindex. Once it has substantial content, switch to index.

---

## GSC sitemap fallback (no service account)

If service account is not yet set up, the user can submit the sitemap manually:
```
GSC → Sitemaps → Add new sitemap → type: sitemap.xml → Submit
```

This covers sitemap submission only. The Indexing API for priority URL crawl requests still requires a service account. Offer to run setup when the user is ready.
