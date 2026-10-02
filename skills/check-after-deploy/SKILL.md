---
name: check-after-deploy
description: Run a site's Relvato monitors right after a deploy and report what passed and what broke, with links. Use after deploying, releasing or merging to production, when the user wants to know the live site still works.
argument-hint: "[site URL or name] [group: flow|security|design|seo|domain|other]"
disable-model-invocation: true
---

# Check the live site after a deploy

Use the Relvato MCP tools (`list_sites`, `site_overview`, `trigger_scan`, `list_runs`, `get_run`). Call Relvato's checks **monitors** when talking to the user.

## 1. Which site

Use the site from `$ARGUMENTS`. Otherwise call `list_sites` and pick the one whose address matches this project's production URL (see `package.json` `homepage`, `CNAME`, `vercel.json`, `netlify.toml` or the README). If it's ambiguous, ask. If the site isn't in Relvato yet, offer `/relvato:setup` instead. If the site isn't ready (ownership not proven), say so and stop.

## 2. Run the monitors

Make sure the deploy has finished: the new version must be live, or the runs test the old one. If you can't tell, ask.

Call `trigger_scan` for the site. If the user named a group, pass `group`; to run one monitor, pass `checkId`. This uses the monthly run quota, so if `notRunOverQuota` lists monitors, tell the user they were skipped.

Tell the user right away what `tellUser` says: how many runs and roughly how many minutes they take. Runs go one after another, so a full run of a site takes several minutes.

## 3. Follow the runs

Don't poll continuously. Check `list_runs` for the site, or `get_run` per runId, about every couple of minutes until every run is `passed`, `failed` or `skipped`. Report each result as it lands, in one line with its run link.

## 4. Report

Summarize briefly:
- **Passed:** the count, plus the names if there are only a few.
- **Failed or found issues:** for each, the monitor, what it found (from `get_run`), and the run link.
- **Skipped:** the reason, for example a firewall challenge or an over-quota run.

If anything failed, offer to fix it with `/relvato:fix-failure <runId>`. Don't accept or ignore any result yourself. If a change looks intended (for example a redesign the visual monitor flagged), ask the user. Only when they confirm, use `accept_visual_change`, `accept_structure_change` or `ignore_finding`; each can be undone.
