---
name: setup
description: Set up Relvato website monitoring for the site this repository deploys — add the site, prove ownership (in the code where possible), pick monitors for what matters, and start the first runs. Use when the user wants to monitor their site, app or store with Relvato.
argument-hint: "[site URL]"
---

# Set up Relvato for this project's site

Use the Relvato MCP tools (`list_sites`, `add_site`, `verify_site`, `start_monitoring_for_goals`, `list_checks`, `add_checks`, `trigger_scan`). Call Relvato's checks **monitors** when talking to the user.

## 1. Find the site's address

Use `$ARGUMENTS` if given. Otherwise look for the production URL in the project: `homepage` in `package.json`, a `CNAME` file, `vercel.json` / `netlify.toml` / `wrangler.toml`, the framework config (`siteUrl`, `metadataBase`, `site:` in Astro), or the README. Don't open `.env` files. If you find none, or several, ask the user which address to monitor. It must be the live public site, not localhost.

## 2. Add the site (or reuse it)

Call `list_sites`. If the site is already there, use it and skip to step 4 when it's ready.

Otherwise call `add_site` with the URL and `platform`: `wordpress` for a WordPress site, `other` for anything else. If the call fails because the user isn't signed in, tell them to run `/mcp`, choose the Relvato server and sign in (there's a free plan), then try again.

## 3. Prove ownership

A new site runs no monitors until ownership is proven; you can't skip this. Read `setup.nextStep` and `setup.domainVerification` from the result.

- **The site's code is in this repo** (not WordPress): offer to add `setup.domainVerification.metaTag` to the `<head>` of the homepage, in the root layout or `index.html`. Show the exact edit and make it only when the user agrees. It works once the change is **deployed**: ask the user to deploy, then call `verify_site`.
- **No access to the code, or the user prefers DNS:** give them the TXT record (`dnsTxtHost`, `dnsTxtValue`) to add at their DNS provider, then call `verify_site`. DNS can take a few minutes to update.
- **WordPress:** the Relvato plugin proves it. Give the user the install step from `setup.nextStep`; they paste the connect token from the site's Relvato page, which you never see. Then call `verify_site`.

If `verify_site` comes back not ready, show its `hint` and wait for the user before retrying.

## 4. Choose monitors

Ask what matters most, for example sign-up or login, checkout, page speed, search visibility or security. Then call `start_monitoring_for_goals` with those goals. Use `list_checks` + `add_checks` instead when the user names specific monitors. Tell the user what was added and anything their plan doesn't include.

## 5. First runs

Ask before starting runs, because they count toward the monthly quota. Then call `trigger_scan` for the site. Tell the user what its `tellUser` says (how long the runs usually take), and don't poll continuously: check `list_runs` every couple of minutes and report results as they finish. For any failure, offer `/relvato:fix-failure`.

Finish with the site's Relvato page link and a one-line summary of what is now monitored.
