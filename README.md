<p align="center">
  <img src="https://www.relvato.com/logo-512.png" alt="Relvato" width="96" height="96">
</p>

<h1 align="center">Relvato MCP server</h1>

<p align="center">
  Website monitoring your AI agent can run: add a site, pick its checks, run them and explain what broke.<br>
  <a href="https://www.relvato.com/developers">Developer docs</a> ·
  <a href="https://registry.modelcontextprotocol.io/v0/servers?search=com.relvato">Official MCP Registry</a> ·
  <a href="https://www.relvato.com">relvato.com</a>
</p>

---

[Relvato](https://www.relvato.com) monitors websites in a real browser. It checks sign-ups, logins, checkout, payments,
page speed, search visibility and security, and re-checks them every time the site changes. It works on any website and
goes deepest on WordPress & WooCommerce.

This is Relvato's **hosted** [Model Context Protocol](https://modelcontextprotocol.io) server. There's nothing to
install or run: point your MCP client at the URL and authenticate with a Relvato API key. This repository holds the
connection details and the registry entry ([`server.json`](server.json)). The server itself runs at Relvato.

| | |
| --- | --- |
| **Endpoint** | `https://app.relvato.com/api/mcp` |
| **Transport** | Streamable HTTP (stateless JSON-RPC over POST) |
| **Auth** | Relvato API key: `Authorization: Bearer rlv_…` (or `x-api-key: rlv_…`) |
| **Registry name** | `com.relvato/relvato` |

## Get an API key

1. [Sign up](https://app.relvato.com/sign-up) (there's a free plan) or sign in.
2. Open **API access** from the account menu, or go to [app.relvato.com/api-access](https://app.relvato.com/api-access).
3. Create a key:
   - **Full access** lets the agent add sites and checks, change schedules and run scans.
   - **Read-only** lets it read sites, runs, health overviews, fix briefs and alert settings. With a read-only key the
     client only sees the read tools.

A key is shown once. Keep it out of files you commit.

## Connect

### Claude Code

```bash
claude mcp add --transport http relvato https://app.relvato.com/api/mcp --header "Authorization: Bearer rlv_your_key"
```

### Cursor

In `~/.cursor/mcp.json` (all projects) or `.cursor/mcp.json` (one project):

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

### VS Code

In `.vscode/mcp.json`. VS Code asks for the key once and stores it securely, so it never sits in the file:

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

### Any other client

Add a remote **HTTP** (streamable HTTP) server with the endpoint above and an `Authorization: Bearer rlv_…` header.
In the common `mcp.json` format:

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

Relvato uses API keys, not OAuth, so a client needs to be able to send a custom header. Clients that accept only a bare
URL can list the tools but can't call them.

## Tools

| Tool | What it does | Key needed |
| --- | --- | --- |
| `list_sites` | List the account's websites and whether each is ready to run checks. | read-only |
| `site_overview` | Plain-language health verdict: what needs attention, every check with its latest run and schedule, and plan usage. | read-only |
| `list_checks` | The checks you can add to a site: what each catches, whether your plan includes it, and which are recommended. | read-only |
| `list_runs` | Recent runs, newest first, optionally for one site. | read-only |
| `get_run` | One run in detail: status, error, warnings, steps, visual comparisons and the dashboard link. | read-only |
| `get_fix_prompt` | For a run that found a problem: the same brief Relvato's own AI answers, to reason about the likely cause and fixes. | read-only |
| `get_alert_settings` | Who is told about what: frequency, severity, each channel's state and routing. No URLs or secrets. | read-only |
| `add_site` | Add a website and get its setup step: connect the WordPress plugin, or verify the domain. | full access |
| `verify_site` | Check that setup step: the plugin connection, or the domain-verification DNS record or meta tag. | full access |
| `add_checks` | Add checks to a site. Each one reports added, already there, or why not. | full access |
| `update_check` | Turn a check on or off, or change its schedule. | full access |
| `trigger_scan` | Run a site's checks now, or a single check, and get the run IDs back. Uses the monthly run quota. | full access |

`initialize` and `tools/list` answer without a key, so directories and clients can show what the server offers. Every
tool call needs one.

## Things to ask

- "Set up monitoring for example.com and tell me what I need to do to connect it."
- "Why did checkout fail on my shop last night, and how do I fix it?"
- "Who gets alerted when a Security check fails, and on which channels?"
- "Run the checks on my site now and tell me if anything broke."

## What an agent can't do

These stay in the Relvato dashboard, with a person looking:

- **Skip ownership.** A new site runs no checks until its WordPress plugin is connected or its domain is verified.
- **Accept changes.** Accepting new visual baselines and ignoring warnings are done in the dashboard.
- **Apply fixes.** No fixes are applied to your site.
- **Touch secrets.** Connect tokens, signing secrets and webhook URLs are never exposed.
- **Go past your plan.** Plan limits apply exactly as in the dashboard.

## Limits

Requests are rate-limited per account, per minute, counted across the REST API and MCP together:

| Plan | Requests per minute |
| --- | --- |
| Free | 30 |
| Pro | 120 |
| Business | 600 |
| Agency | 2,400 |

Runs started with `trigger_scan` count toward the same monthly run quota as scheduled checks. See
[pricing](https://www.relvato.com/pricing).

## Also available

- **REST API.** The same data at `https://app.relvato.com/api/v1`, using the same keys. See the
  [developer docs](https://www.relvato.com/developers).
- **Webhooks.** A signed JSON event for every alert (Pro and up).

## Support

- Questions and bug reports: [relvato.com/contact](https://www.relvato.com/contact), or open an issue here.
- Service status: [relvato.com/status](https://www.relvato.com/status).
- Security issues: please use the contact page, not a public issue.

This repository contains documentation and configuration only. The Relvato service is proprietary and covered by its
[terms](https://www.relvato.com/terms). The contents of this repository are MIT-licensed (see [LICENSE](LICENSE)), so
copy the config snippets freely.
