# Installing the Relvato MCP server (for AI agents)

Relvato's MCP server is **hosted**. There is nothing to clone, build, or install, and no command or package to run.
Setup is one config entry pointing at a URL. The user then either signs in or gives you an API key:

- **Your client supports MCP sign-in (OAuth)**, as Claude, Claude Code, ChatGPT, VS Code and Cursor do: add the URL
  alone and let the client start the sign-in. Relvato's sign-in opens in the browser; the user allows access (and picks
  a workspace if they're in an organization). Skip step 1.
- **Otherwise** (Cline, scripts), use an API key: follow steps 1 to 3.

If the user belongs to an organization and wants their **personal** workspace, use an API key; sign-in only offers the
organization's workspaces.

## 1. Get the user's API key

Ask the user for their Relvato API key. It starts with `rlv_`.

If they don't have one, tell them to:

1. Sign in or sign up (there's a free plan) at https://app.relvato.com
2. Open **API access** from the account menu, or go to https://app.relvato.com/api-access
3. Click **Create API key**. Choose **full access** if you should be able to add sites and run scans; **read-only** only
   lets you read results.

Never guess a key, and never write one into a file inside a git repository.

## 2. Add the server

Add this to the MCP settings file, replacing `rlv_your_key` with the user's key.

**Cline** (`cline_mcp_settings.json`):

```json
{
  "mcpServers": {
    "relvato": {
      "type": "streamableHttp",
      "url": "https://app.relvato.com/api/mcp",
      "headers": {
        "Authorization": "Bearer rlv_your_key"
      },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

Other clients that use the `mcp.json` format take the same entry with `"type": "http"` instead of `"streamableHttp"`.
Merge it into the existing `mcpServers` object; don't replace the other servers.

## 3. Check it works

Call the `list_sites` tool.

- **A list, possibly empty:** the setup worked. If it's empty, offer to add the user's first site with `add_site`.
- **401 / unauthorized:** the key is wrong or has been revoked. Ask the user to check it or create a new one.
- **"This API key is read-only":** the key can't make changes. Reads work; for anything else the user needs a
  full-access key.

## Notes

- Transport is streamable HTTP over POST, stateless. The server doesn't use SSE or a long-lived connection.
- A new site runs no checks until its owner connects the WordPress plugin or verifies the domain. `add_site` returns the
  exact step; relay it to the user, because you can't do it for them.
- Runs started with `trigger_scan` use the account's monthly run quota.
- Docs: https://www.relvato.com/developers
