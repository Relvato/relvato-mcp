<p align="center">
  <img src="https://www.relvato.com/logo-512.png" alt="Relvato" width="96" height="96">
</p>

<h1 align="center">Relvato MCP server</h1>

<p align="center">
  Website monitoring your AI agent can run: add a site, set up and tune its monitors, run them, explain what broke and fix it with you.<br>
  <a href="https://www.relvato.com/developers">Developer docs</a> ·
  <a href="https://registry.modelcontextprotocol.io/v0/servers?search=com.relvato">Official MCP Registry</a> ·
  <a href="https://www.relvato.com">relvato.com</a>
</p>

---

[Relvato](https://www.relvato.com) monitors websites in a real browser. It checks sign-ups, logins, checkout, payments,
page speed, search visibility and security, and re-checks them every time the site changes. It works on any website and
goes deepest on WordPress & WooCommerce.

This is Relvato's **hosted** [Model Context Protocol](https://modelcontextprotocol.io) server. There's nothing to
install or run: point your MCP client at the URL and **sign in to Relvato**, or use a Relvato API key. This repository
holds the connection details and the registry entry ([`server.json`](server.json)). The server itself runs at Relvato.

| | |
| --- | --- |
| **Endpoint** | `https://app.relvato.com/api/mcp` |
| **Transport** | Streamable HTTP (stateless JSON-RPC over POST) |
| **Auth** | Sign in (OAuth 2.1: PKCE, Client ID Metadata Documents, resource indicators), or an API key: `Authorization: Bearer rlv_…` |
| **Registry name** | `com.relvato/relvato` |

## Sign in (no key needed)

Sign-in works in clients that identify themselves with a **Client ID Metadata Document** (CIMD): Claude (claude.ai,
Desktop, and Claude Code 2.1.81 or later), ChatGPT and VS Code. Relvato doesn't offer dynamic client registration, so a
client that can only register itself that way (Cursor for now, Docker's MCP gateway) uses an
[API key](#or-use-an-api-key) instead.

In a client that supports it, add the endpoint with nothing else. The first time the client uses it, it opens Relvato's sign-in:

1. [Sign up](https://app.relvato.com/sign-up) (there's a free plan) or sign in.
2. Allow the client to **read your sites and results** and to **make changes**: add and configure sites and monitors, change schedules
   and run scans.
3. If you belong to an organization, pick which of its workspaces to connect. **Your personal workspace isn't offered
   there, so use an API key for it.**

Discovery follows the MCP authorization spec. A 401 points to
`/.well-known/oauth-protected-resource/api/mcp`, and the authorization server is `https://clerk.relvato.com`
(`client_id_metadata_document_supported`; no registration endpoint). The scopes are `relvato:read`, `relvato:write` and
`user:org:read`.

## Or use an API key

For scripts, for clients that can't sign in (no CIMD support), or for your personal workspace while you're in an organization:

1. Open **API access** from the account menu, or go to [app.relvato.com/api-access](https://app.relvato.com/api-access).
2. Create a key:
   - **Full access** lets the agent add and configure sites and monitors, run them, review results and apply the fixes
     runs propose.
   - **Read-only** lets it read everything: sites, health and uptime, performance, runs, monitors, WordPress updates
     and activity, fix briefs and alert settings. With a read-only key the client only sees the read tools.
3. Send it as `Authorization: Bearer rlv_…` (or `x-api-key: rlv_…`). A key is shown once. Keep it out of files you
   commit.

## Connect

### Claude (claude.ai, Desktop, mobile)

**Settings → Connectors → Add custom connector**. Enter the URL `https://app.relvato.com/api/mcp`, then **Connect** and
sign in.

### Claude Code

```bash
claude mcp add --transport http relvato https://app.relvato.com/api/mcp
```

Then run `/mcp`, choose **relvato**, then **Authenticate**. With an API key instead:

```bash
claude mcp add --transport http relvato https://app.relvato.com/api/mcp --header "Authorization: Bearer rlv_your_key"
```

### Cursor

In `~/.cursor/mcp.json` (all projects) or `.cursor/mcp.json` (one project), with an API key. Cursor doesn't support
Client ID Metadata Documents yet, so it can't use Relvato's sign-in:

```json
{
  "mcpServers": {
    "relvato": {
      "url": "https://app.relvato.com/api/mcp",
      "headers": {
        "Authorization": "Bearer rlv_your_key"
      }
    }
  }
}
```

Once Cursor supports CIMD, the URL alone will be enough: Cursor will offer to sign in.

### VS Code

In `.vscode/mcp.json`. VS Code offers to sign in:

```json
{
  "servers": {
    "relvato": {
      "type": "http",
      "url": "https://app.relvato.com/api/mcp"
    }
  }
}
```

With an API key, VS Code can ask for it once and store it securely, so it never sits in the file:

```json
{
  "inputs": [
    { "type": "promptString", "id": "relvato-key", "description": "Relvato API key (rlv_…)", "password": true }
  ],
  "servers": {
    "relvato": {
      "type": "http",
      "url": "https://app.relvato.com/api/mcp",
      "headers": {
        "Authorization": "Bearer ${input:relvato-key}"
      }
    }
  }
}
```

### Gemini CLI

Install the extension from this repository:

```bash
gemini extensions install https://github.com/Relvato/relvato-mcp
```

Then run `/mcp auth relvato` to sign in. If the sign-in doesn't open (it needs CIMD support in the Gemini CLI), use an
API key: add the server to `~/.gemini/settings.json`:

```json
{
  "mcpServers": {
    "relvato": {
      "httpUrl": "https://app.relvato.com/api/mcp",
      "headers": {
        "Authorization": "Bearer rlv_your_key"
      }
    }
  }
}
```

### Cline

In `cline_mcp_settings.json` (MCP Servers → Configure), with an API key:

```json
{
  "mcpServers": {
    "relvato": {
      "type": "streamableHttp",
      "url": "https://app.relvato.com/api/mcp",
      "headers": {
        "Authorization": "Bearer rlv_your_key"
      }
    }
  }
}
```

Agents that set servers up themselves can follow [`llms-install.md`](llms-install.md).

### Any other client

Add a remote **HTTP** (streamable HTTP) server with the endpoint above. If the client supports MCP sign-in with Client
ID Metadata Documents (CIMD), that's all. Otherwise (including clients that only do dynamic client registration) add an
`Authorization: Bearer rlv_…` header. In the common `mcp.json` format:

```json
{
  "mcpServers": {
    "relvato": {
      "type": "http",
      "url": "https://app.relvato.com/api/mcp",
      "headers": {
        "Authorization": "Bearer rlv_your_key"
      }
    }
  }
}
```

## Tools

52 tools, grouped by job in the order `tools/list` sends them. **read** tools work with any connection; **full access** tools need a full-access key, or a sign-in that allowed changes.

### Set up

| Tool | What it does | Access needed |
| --- | --- | --- |
| `list_sites` | List the account's websites and whether each is ready to run monitors. | read |
| `add_site` | Add a website and get its setup step: connect the WordPress plugin, or verify the domain. | full access |
| `verify_site` | Check that setup step: the plugin connection, or the domain-verification DNS record or meta tag. | full access |
| `start_monitoring_for_goals` | Pick what matters (checkout, security, search, speed, …) and get the monitors that cover it on your plan. | full access |
| `list_checks` | The monitors you can add to a site: what each catches, whether your plan includes it, and which are recommended. | read |
| `add_checks` | Add monitors to a site; each one reports added, already there, or why not. | full access |
| `dismiss_recommendation` | Stop recommending a monitor for a site, or bring dismissed recommendations back. | full access |
| `calibrate_site` | WooCommerce: read the store again (a product to buy, checkout type, currency) after it changed. | full access |
| `verify_vitals_beacon` | Check the real-user Web Vitals beacon is installed. | full access |

### Read

| Tool | What it does | Access needed |
| --- | --- | --- |
| `site_overview` | Plain-language health verdict: what needs attention, every monitor with its latest run and schedule, plan usage and, on WordPress, the site's AI abilities. | read |
| `get_site_health` | Pass rate now and over 30 days, the trend over 28–182 days, each group's briefing, and probe uptime with incidents. | read |
| `get_performance` | Lab and real-user Core Web Vitals, what slows the site down, and the top fixes with steps. | read |
| `get_check` | One monitor in full: schedule, its own settings (pages, masks, checkout details, …), ignored findings, its recipe if AI-written, latest run. | read |
| `list_runs` | Recent runs, newest first — by site, monitor or status, with paging. | read |
| `get_run` | One run in detail: steps, findings, visual comparisons with image links, the fix it proposes, plugins it can update. | read |
| `get_fix_prompt` | For a run that found a problem: the same brief Relvato's own AI answers, to reason about the likely cause and fixes. | read |
| `get_updates` | WordPress: auto-update windows and what they updated or rolled back, held versions, safe updates, hardening. | read |
| `get_safe_update` | Follow a safe plugin update or deactivation until it passes or is undone. | read |
| `get_quarantine` | Quarantine mode: available or not, on until when, every change it saw, past sessions. | read |
| `get_activity_log` | WordPress: sign-ins, failed sign-ins, accounts, updates, settings and file edits, within your plan's history. | read |
| `get_notifications` | The dashboard's notifications: failing monitors, plugin or beacon problems, paused channels, billing. | read |
| `get_site_settings` | A site's settings — what an agent may change and what stays in the dashboard (never secret values). | read |
| `get_alert_settings` | Who is told about what: frequency, severity, each channel's state and routing — no URLs or secrets. | read |
| `get_status_pages` | Status pages and client reports: settings only, never a private link or recipients' addresses. | read |

### Tune

| Tool | What it does | Access needed |
| --- | --- | --- |
| `update_check` | Turn a monitor on or off, or change when it runs: schedule, re-runs on WordPress updates, random extra runs. | full access |
| `update_visual_monitor` | Add pages of your own by address to the visual monitor, set masks, devices, browsers and the threshold. | full access |
| `update_check_settings` | A monitor's own settings: checkout details, extra pages to scan, what the site's AI feature must answer, DKIM selectors, … | full access |
| `add_custom_check` | Describe what must be true on a page; Relvato's AI writes the monitor. | full access |
| `edit_custom_check` | Give a custom monitor a new goal or page; the AI writes a new recipe to approve. | full access |
| `reauthor_custom_check` | Answer the AI's question, or have it write a custom monitor's recipe again. | full access |
| `approve_custom_check` | Approve a proposed recipe so the custom monitor runs (interactive recipes need an explicit OK). | full access |
| `update_site_settings` | Pacing, spacing between runs, firewall retry, flaky-monitor recovery, plugin rollback. | full access |
| `update_alert_settings` | Alert timing and minimum severity. Who receives alerts and muting stay in the dashboard. | full access |

### Run and act

| Tool | What it does | Access needed |
| --- | --- | --- |
| `trigger_scan` | Run a site's monitors now — all, one group, or one monitor — and get the run ids back (uses the monthly quota). | full access |
| `cancel_queued_runs` | Stop a site's queued runs; the one already going finishes. | full access |
| `request_ai_fix` | Ask Relvato's AI for a fix suggestion for a failed run (paid plans). | full access |
| `apply_fix` | WordPress: apply the one-click fix a run proposes — only that fix — then re-run the monitor. | full access |
| `start_safe_update` | WordPress: update or deactivate a vulnerable plugin, re-check, and undo it if anything breaks. | full access |
| `start_quarantine` | After a hack: hourly security runs for 48 hours with plugin updates paused. | full access |
| `extend_quarantine` | Keep quarantine mode on for another 48 hours. | full access |
| `send_test_alert` | Send a test alert by email, Slack or webhook. | full access |
| `resume_webhook` | Turn webhook alerts back on after Relvato paused them for failed deliveries. | full access |
| `report_false_positive` | Tell Relvato's team a result is wrong; it changes nothing on the run. | full access |

### Review results

| Tool | What it does | Access needed |
| --- | --- | --- |
| `ignore_finding` | Stop a finding from counting on a monitor (undoable). Only when you confirm it's expected. | full access |
| `unignore_finding` | Make an ignored finding count again. | full access |
| `accept_visual_change` | Make this run's screenshot the new baseline for a page (undoable). | full access |
| `ignore_visual_change` | Ignore the area that changed on a page from now on (undoable). | full access |
| `flag_visual_change_as_problem` | Mark a visual change the AI let pass as a failure. | full access |
| `undo_visual_review` | Undo the last visual accept or ignore. | full access |
| `accept_structure_change` | Adopt a page's new structure as its baseline (undoable). | full access |
| `ignore_structure_change` | Ignore specific element changes on a page (undoable). | full access |
| `undo_structure_review` | Undo the last structure accept or ignore. | full access |

`initialize` and `tools/list` answer without signing in, so directories and clients can show what the server offers.
Every tool call needs a sign-in or a key. A read-only key, or a sign-in without write access, gets only the read tools.

## Things to ask

- "Set up monitoring for example.com and tell me what I need to do to connect it."
- "Add my pricing and blog pages to the visual monitor, and run it."
- "Why did checkout fail on my shop last night, and how do I fix it?" (and, once you agree, "apply that fix")
- "How fast is my shop for real visitors, and what's slowing it down?"
- "The homepage redesign is intentional: accept the new screenshots."
- "Who signed in to WordPress this week, and did anyone fail to?"
- "Write a monitor that checks the Pro plan still shows a price."

## What an agent can't do

The agent works under the same rules as the dashboard. Some things stay in the dashboard, with a person looking:

- **Skip ownership.** A new site runs no monitors until its WordPress plugin is connected or its domain is verified.
- **Silence something for good.** It can accept or ignore a result only in ways that can be undone, and only when you
  confirm the change is intended. Approving file-integrity, script and DNS changes (no undo; exactly what an attacker
  wants clicked) stays in the dashboard.
- **Change your site beyond what a run proposes.** On WordPress it applies only the one-click fix a run proposes, and
  updates or deactivates a vulnerable plugin only with automatic undo. Destructive tools are marked, so your client
  asks you first.
- **Weaken alerts or protection.** Who receives alerts, muting, stopping quarantine mode, the proxy and the firewall
  token stay in the dashboard.
- **Touch secrets.** Connect tokens, signing secrets, webhook URLs and report share links are never exposed.
- **Delete anything, or change billing.**
- **Go past your plan.** Plan limits apply exactly as in the dashboard.

## Limits

Requests are rate-limited per workspace, per minute, counted across the REST API and MCP together, whether you sign in
or use a key:

| Plan | Requests per minute |
| --- | --- |
| Free | 30 |
| Pro | 120 |
| Business | 600 |
| Agency | 2,400 |

Runs started with `trigger_scan` (and the runs that confirm `apply_fix` and `start_safe_update`) count toward the same
monthly run quota as scheduled runs. See
[pricing](https://www.relvato.com/pricing).

## Also available

- **REST API.** The same read data at `https://app.relvato.com/api/v1` (sites, health, performance, runs, monitors,
  updates, activity, notifications), using the same keys. See the [developer docs](https://www.relvato.com/developers).
- **Webhooks.** A signed JSON event for every alert (Pro and up).

## Support

- Questions and bug reports: [relvato.com/contact](https://www.relvato.com/contact), or open an issue here.
- Service status: [relvato.com/status](https://www.relvato.com/status).
- Security issues: please use the contact page, not a public issue.

This repository contains documentation and configuration only. The Relvato service is proprietary and covered by its
[terms](https://www.relvato.com/terms). The contents of this repository are MIT-licensed (see [LICENSE](LICENSE)), so
copy the config snippets freely.
