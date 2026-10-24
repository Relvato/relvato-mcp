---
name: search-traffic
description: Explain a website's search traffic and SEO health from Relvato's monitors — Google Search Console and Bing Webmaster Tools clicks, impressions, click-through rate and average position against the site's usual, pages in the index, the pages and queries losing ground, crawl errors and sitemaps — plus SEO integrity (noindex, robots.txt, canonicals), redirects and readiness for AI answer engines. Use when the user asks about their Google or Bing traffic, rankings, indexing, a drop in search, or SEO.
argument-hint: "[site URL or name] [google|bing]"
---

# Search traffic and SEO

Use the Relvato MCP tools (`list_sites`, `list_checks`, `list_runs`, `get_run`, `start_monitoring_for_goals`, `trigger_scan`). Call Relvato's checks **monitors** when talking to the user.

## 1. Which site

Use `$ARGUMENTS` if it names a site; otherwise call `list_sites` and ask if there are several. If the tools aren't available or the user isn't signed in, tell them to connect Relvato and sign in (in Claude: Settings → Connectors; in Claude Code: run `/mcp`).

## 2. Find the search monitors

Call `list_checks` for the site. Relvato's SEO / AEO group has:

- `google-search` (Google Search Console) and `bing-search` (Bing Webmaster Tools): the engines' own numbers, read once a day
- `seo-guard` (SEO integrity: noindex, robots.txt, canonicals, titles), `sitemap-health`, `redirect-health`, `i18n-health` (hreflang) and `aeo-ready` (AI answer engines)

If none of these is added, offer to add them with `start_monitoring_for_goals` and the goal `search`, and only with the user's OK.

## 3. Google or Bing not connected yet

The engines' monitors need the site's owner to connect Google Search Console or Bing Webmaster Tools once, in the Relvato dashboard (the site's Settings → Search engines). That can't be done from here. When a run was skipped because the engine isn't connected, say so, give the user the site's Relvato link, and mention that the access is read-only.

## 4. Read the latest results

Call `list_runs` with the site and the monitor's `checkId`, take the newest run, then call `get_run`. A search run carries its card in `metrics.search`:

- `tiles`: clicks, impressions, click-through rate, average position (Bing gives it per week), pages in Google's results or in Bing's index, and Bing's crawl numbers, each against the site's usual
- `dailies`: each day's value and its usual
- `losers`: the pages and queries that lost the most ground (Google: the last 7 days against an average week; Bing: its latest week against the 4 before)
- `inspections` (Google's URL Inspection) and `bingPages` (Bing's last crawl): the site's key pages
- `crawl`, `crawlIssues` and `sitemaps`, and `notInApi`: what that engine's API doesn't offer

The run's findings say what regressed and, for a drop, what changed on the site shortly before. Read `seo-guard`, `sitemap-health` and `redirect-health` runs the same way when they failed or warned.

## 5. Explain it

- Lead with the verdict and the dates the numbers cover: both engines report a day or more behind.
- Compare with the site's usual, not with yesterday. Each day is compared with the same weekday before it.
- Name the pages and queries behind a change.
- Before a drop, say what changed on the site (updates, deploys, SEO integrity findings). Present it as timing, not proven cause.
- When clicks are too few to judge, say that impressions are judged instead.
- Say what an engine doesn't report: Google's API has no index count, and Bing gives position only per week.

## 6. Next steps

Offer what fits:
- Run the search monitors now with `trigger_scan` and `group: "seo"`. Ask first: runs count toward the monthly quota, and the engines' numbers change only once a day.
- Use the `fix-failure` skill when an SEO integrity finding points at the site's code: a noindex, a robots.txt rule or a canonical.
- Add the SEO monitors that are missing.
