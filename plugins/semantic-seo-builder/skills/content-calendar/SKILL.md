---
name: content-calendar
description: Builds a 3-month content calendar for a Semantic SEO local business site, maps every article to the topical map, and optionally sets up automated scheduled tasks. Use when user says content calendar, content schedule, plan my content, 3 month plan, schedule my posts, automate my content, or when should I publish.
tools:
  - Read
  - Write
---

# Content Calendar

This skill does two things:
1. Builds a 3-month editorial calendar based on the topical map
2. Sets up automated scheduled tasks for the weekly pipeline and monthly audit

## Step 1 — Read the topical map

Read `03-topical-map.md` for the full list of Tier 2 (subtopic) and Tier 3 (supporting content) pages.
Read `weekly-log.md` if it exists to see what's already been published.
Read `01-foundation.md` for any seasonal opportunities in this niche.

If none exist, ask: "What niche is your site in? Roughly how many articles are you planning to write?"

---

## Step 2 — Build the 3-month calendar

Assign every unpublished Tier 2 and Tier 3 page to a specific week across the next 3 months.

Scheduling rules:
- **Week 1–4**: Focus on Tier 2 subtopic pages that support P1 service pages (authority foundation)
- **Week 5–8**: Start Tier 3 supporting articles for the primary pillar
- **Week 9–12**: Tier 3 supporting articles for secondary pillars + seasonal content
- **Monthly**: One "content refresh" slot — update an existing page rather than writing new content
- **Seasonal**: If the niche has seasonal patterns (e.g., a heating company publishing "winter boiler checks" content in September), schedule those 6–8 weeks in advance

Produce a calendar table:

| Week | Date (Monday) | Content Type | Topic/Title | Pillar | Priority | Status |
|------|--------------|-------------|-------------|--------|----------|--------|
| Week 1 | [date] | Subtopic page | [title] | [pillar] | P2 | Pending |
| Week 2 | [date] | Supporting article | [title] | [pillar] | P3 | Pending |
| Week 3 | [date] | Supporting article | [title] | [pillar] | P3 | Pending |
| Week 4 | [date] | Content refresh | [page to update] | — | P1 | Pending |
| ... | | | | | | |

Also produce a monthly actions column:

| Month | Monthly Audit | Key Focus | GBP Campaign |
|-------|--------------|-----------|-------------|
| Month 1 | End of month | Build primary pillar | Promote main service |
| Month 2 | End of month | Build secondary pillars | Seasonal promotion |
| Month 3 | End of month | Fill gaps, refresh thin content | New service highlight |

---

## Step 3 — Save the calendar

Write the calendar to `content-calendar.md`. Include the full table plus:

**Publishing checklist for each week:**
- [ ] Run `/weekly-pipeline` on Monday to write the article
- [ ] Publish to WordPress the same day
- [ ] Add the return internal link from the parent page
- [ ] Publish GBP post on Wednesday (mid-week gets more engagement)
- [ ] Update `weekly-log.md`

**Monthly checklist:**
- [ ] Run `/monthly-audit` on the last Monday of the month
- [ ] Implement all "refresh" tasks from the audit report
- [ ] Review and update the content calendar based on new opportunities found

---

## Step 4 — Set up automated scheduling

Ask the user: "Do you want me to set up automatic reminders so I run the weekly pipeline and monthly audit for you on a schedule? I can run the weekly pipeline every Monday and the monthly audit at the end of each month."

If yes, set up the following scheduled tasks:

**Weekly pipeline — every Monday:**
Set up a scheduled task with:
- Schedule: every Monday at 9:00 AM (user's timezone)
- Task prompt: "Run the /weekly-pipeline skill. Read 03-topical-map.md and weekly-log.md to pick the next topic in the queue. Write the full article and GBP post. After the article is published to WordPress, run /indexing in weekly mode to submit the URL to Google and log the crawl check result. Save the weekly log."

**Monthly audit — last Monday of each month:**
Set up a scheduled task with:
- Schedule: monthly (on the 28th of each month to catch month-end regardless of month length)
- Task prompt: "Run the /monthly-audit skill. Read all phase files and the weekly log. Produce the monthly report with ranking checks, content audit, and next-month action plan."

Tell the user: "I've set up the automated schedule. Every Monday I'll run the weekly pipeline and prepare your content. Every month-end I'll audit your site and report back. You just need to copy the articles into WordPress — I'll do the writing and strategy."

---

## What the full ongoing rhythm looks like

**Every Monday (automated):**
→ `/weekly-pipeline` runs → article + GBP post produced → user publishes to WordPress

**Every Wednesday:**
→ User publishes GBP post from Monday's output

**Every last Monday of the month (automated):**
→ `/monthly-audit` runs → report produced → user implements refresh tasks

**Quarterly (manual):**
→ Run `/semantic-research` again to check for new entities and attribute shifts
→ Run `/topical-map` to check if the topical map needs expanding
→ Update `content-calendar.md` for the next quarter
