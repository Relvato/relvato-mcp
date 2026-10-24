---
name: security-review
description: Review a website's security from Relvato's monitors — known-vulnerable plugins, themes and core, signs of a hack (spam pages, injected redirects, a listing as unsafe), changed or unexpected files, new admin users and suspicious logins, exposed backups and config files, new third-party scripts, SSL and DNS — and say what to do first. Use when the user asks whether their site is secure or hacked, or about vulnerabilities, malware or a security alert.
argument-hint: "[site URL or name]"
---

# Review a site's security

Use the Relvato MCP tools (`list_sites`, `site_overview`, `list_checks`, `list_runs`, `get_run`, `get_updates`, `get_quarantine`, `start_safe_update`, `get_safe_update`, `start_quarantine`, `trigger_scan`). Call Relvato's checks **monitors** when talking to the user.

## 1. Which site

Use `$ARGUMENTS` if it names a site; otherwise call `list_sites` and ask if there are several. If the tools aren't available or the user isn't signed in, tell them to connect Relvato and sign in (in Claude: Settings → Connectors; in Claude Code: run `/mcp`).

## 2. Read the security monitors

Call `site_overview` for what needs attention. Call `list_checks` to see which security monitors the site has, then `list_runs` and `get_run` for each one whose latest run failed or warned. Relvato's Security group:

- `vuln-scan`: known vulnerabilities in the site's plugins, themes and WordPress core
- `hack-symptoms`: signs of a hack, such as spam pages, injected redirects or links, or a listing as unsafe
- `file-integrity` and `theme-integrity`: core, plugin and theme files that changed, or files that shouldn't be there
- `user-audit` and `admin-logins`: new or elevated admin users, and suspicious sign-ins
- `exposure-scan`: backups, debug logs and config files anyone can download
- `script-origins`: new third-party scripts on the pages, a card skimmer's usual way in
- `mixed-content`, `wp-health`, `backup-freshness` and `async-health`: hygiene

The Domain group's SSL, DNS and email-record monitors matter here too. On WordPress, call `get_updates` for vulnerable plugins with an update and any rollbacks, and `get_quarantine` for a recent cleanup watch.

## 3. Report

Order by urgency:

1. Signs of an active compromise: hack symptoms, unexpected file changes, unknown admins, new scripts.
2. Known vulnerabilities, and whether a fixed version exists.
3. Exposed files.
4. Hygiene.

A finding is a sign, not proof. Say what Relvato saw, where, and since when, and link each run. If nothing is wrong, say which monitors are watching, so "all clear" means something.

If the site has no security monitors, offer to add them with `start_monitoring_for_goals` and the goal `security`, and only with the user's OK.

## 4. What to do

- **File-integrity, script-origin and DNS changes** are approved by the owner in the Relvato dashboard, never from here. Say what to look at and link the site.
- **Vulnerable plugin on WordPress:** with the user's OK, `start_safe_update` updates the plugin (or deactivates it) and re-runs the monitors that were passing. With the Relvato plugin 1.77.0 or later, an administrator applies the change in WordPress from the link it returns, and Relvato checks the result and says if anything broke. Follow it with `get_safe_update`. It counts as runs.
- **What looks like an active hack:** change admin and hosting passwords, remove unknown admins, restore or clean the files, then run the security monitors again with `trigger_scan` and `group: "security"`. Ask first, because runs count toward the monthly quota. After a cleanup, offer Quarantine mode (WordPress, paid plans): `start_quarantine` runs the security monitors every hour for 48 hours and alerts on every unexpected change. Ask first; `get_quarantine` says how many runs it uses.
- **Cause in the site's code**, such as an injected script in a template: offer the `fix-failure` skill.
