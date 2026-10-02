---
name: fix-failure
description: Diagnose a failed Relvato run and fix the cause in this repository's code — read the run's steps, findings and fix brief, find what changed, make the smallest fix, then re-run that one monitor after the deploy. Use when a Relvato monitor failed or found a problem, or the user asks why the site broke.
argument-hint: "[run id or run URL]"
---

# Fix a failed Relvato run in the code

Use the Relvato MCP tools (`list_runs`, `get_run`, `get_fix_prompt`, `get_check`, `trigger_scan`). Call Relvato's checks **monitors** when talking to the user.

## 1. Find the run

Use the run id from `$ARGUMENTS`; a run URL ends with `/runs/<id>`. Otherwise call `list_runs` with `status: "failed"` for this project's site, take the newest, and confirm it with the user if there's more than one recent failure.

## 2. Understand it

Call `get_run`. Read the steps (where it stopped), the findings and warnings, any visual comparisons, and the fix it proposes. Then call `get_fix_prompt` for Relvato's diagnosis brief. If you need the monitor's own settings (pages, selectors, checkout details), call `get_check` with the run's `checkId`.

Tell the user in two or three sentences what broke, where, and the likely cause.

## 3. Find the cause in the code

Look in this repository:
- **What changed:** `git log` between the monitor's last passing run and this failure (the times are in `get_run` / `list_runs`), then `git diff` for those commits in the files involved.
- **Where to look:** search for what the run touched: the route or page, the form fields, button or link text, selectors, API endpoints, redirects, headers or the script it flagged.

If the cause isn't in this code, say so and point to where it is likely: hosting or CDN config, DNS, a third-party script, the CMS or a plugin. Don't change code to hide it.

## 4. Fix it

Propose the smallest change that fixes the cause, show it, and make it only when the user agrees. Run the project's own tests or build if there are any.

Never weaken or delete what the monitor checks to make it pass. If the failure is an **intended** change (a renamed button, a redesigned page), tell the user. With their confirmation, update the monitor instead (`update_check_settings` / `update_visual_monitor`), or accept the change (`accept_visual_change` / `accept_structure_change`, both undoable).

On a **WordPress** site whose run proposes a one-click fix (in `get_run`), you can offer `apply_fix` instead of editing code. It applies only that proposed fix through the Relvato plugin and can be undone; describe what it changes and get the user's go-ahead first.

## 5. Confirm on the live site

After the fix is deployed, call `trigger_scan` with the site and the monitor's `checkId` to re-run just that monitor. Tell the user it usually takes about the time `tellUser` gives. Then check `get_run` about once a minute until it finishes, and report whether it now passes, with the run link.
