---
name: site-health
description: Report how a website monitored by Relvato is doing across all its monitors — uptime and SSL, checkout and logins, security, speed, SEO and search traffic, visual changes and broken links, domain and DNS, legal compliance — and what needs attention first. Use when the user asks how their site is doing, whether anything is broken, or for a status report on one or all of their sites.
argument-hint: "[site URL or name]"
---

# Report on a site's health

Use the Relvato MCP tools (`list_sites`, `site_overview`, `get_site_health`, `list_runs`, `get_run`). Call Relvato's checks **monitors** when talking to the user. This only reads results: don't start runs unless the user asks, because runs count toward the account's monthly quota.

## 1. Which site

Use `$ARGUMENTS` if it names a site (its address or name). Otherwise call `list_sites`: with one site, use it; with several, ask which one, or give one line per site and go deeper on those that need attention.

If the tools aren't available or answer that the user isn't signed in, tell them to connect Relvato and sign in (in Claude: Settings → Connectors; in Claude Code: run `/mcp`). There's a free plan.

## 2. Read it

Call `site_overview` for the verdict: what needs attention (real issues, and findings that are likely false positives), each monitor with its latest run, the setup state, and recommended monitors not yet added. Call `get_site_health` for the share of monitors passing, the trend, and each monitoring group's briefing. For a failed run you need to explain, call `get_run`.

## 3. Report

Lead with one sentence: is the site fine, and if not, the most important problem. Then go through Relvato's monitoring groups in this order, skipping groups the site has no monitors in:

- **Uptime**: uptime, heartbeats, ports
- **User flows**: checkout and payments, login, sign-up, password reset, forms, order and account emails
- **Security**: vulnerable plugins and themes, signs of a hack, changed files, admin users and logins, exposed files, third-party scripts
- **Performance**: Core Web Vitals in the lab and from real visitors
- **SEO / AEO**: SEO integrity, sitemaps, redirects, Google Search Console and Bing Webmaster Tools numbers, AI answer engines
- **Design**: visual changes, page structure, broken links, errors
- **Domain**: SSL certificate, domain renewal, DNS, email records
- **Legal compliance**: legal pages, cookie consent, accessibility

For each group with something to look at, say what's wrong, since when, and link the run. When `site_overview` marks a finding as a likely false positive, say so plainly. Don't list passing monitors one by one: "6 monitors passing" is enough.

If a group the user cares about has no monitors, say so and name what `site_overview` recommends. Add monitors only when the user agrees (`start_monitoring_for_goals` or `list_checks` + `add_checks`).

## 4. Next steps

Offer what fits: the details of a failed run, the `fix-failure` skill when the cause is likely in the site's code, the `search-traffic` skill for search, or the `security-review` skill for security. End with the site's Relvato link.
