# Relvato

The `relvato` MCP server is Relvato's hosted website monitoring (https://www.relvato.com). It uses the Relvato API key
the user entered when installing this extension. If tools answer 401, the key is wrong or was revoked: ask the user to
create a new one at https://app.relvato.com/api-access and set it with `gemini extensions config relvato`.

- Start with `list_sites`, then `site_overview` for a plain-language verdict on one site.
- To explain a failure: `list_runs` → `get_run` → `get_fix_prompt`.
- Runs started with `trigger_scan` use the account's monthly quota; say so before starting many. A site's runs go one
  at a time, so tell the user what `tellUser` says about how long they take, and check back every couple of minutes
  (`list_runs` / `get_run`) instead of continuously.
- Tools marked destructive (`apply_fix`, `start_safe_update`, accept/ignore results) change the user's site or what
  counts as a problem. Ask the user first, every time, and only accept or ignore a change they confirm is intended.
- Relvato calls them "monitors", not "checks" or "journeys", when talking to the user.
