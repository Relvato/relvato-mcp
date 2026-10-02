# Relvato

The `relvato` MCP server is Relvato's hosted website monitoring (https://www.relvato.com). Sign in with `/mcp auth relvato`
(OAuth; there is a free plan). An API key works too: see the README.

- Start with `list_sites`, then `site_overview` for a plain-language verdict on one site.
- To explain a failure: `list_runs` → `get_run` → `get_fix_prompt`.
- Runs started with `trigger_scan` use the account's monthly quota. Say so before starting many.
- Tools marked destructive (`apply_fix`, `start_safe_update`, accept/ignore results) change the user's site or what
  counts as a problem. Ask the user first, every time, and only accept or ignore a change they confirm is intended.
- Relvato calls them "monitors", not "checks" or "journeys", when talking to the user.
