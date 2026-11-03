# Installing the Domainee MCP server

These instructions are for an AI coding agent (Cline, Claude Code, Cursor) that is
installing this server for a user. The server is hosted: there is nothing to clone,
build or run.

## What you need from the user

Nothing, to start. The 18 free diagnostic tools (SSL, DNS, WHOIS and similar) work with
no key.

For the 15 workspace tools (connecting and managing customer domains, webhooks, origin
secret), ask the user for a Domainee API key. They create one at
https://domainee.dev/developers. Keys start with `sk_live_`. Ask the user to paste it;
never invent one.

## Configuration

Add this to the MCP settings file. In Cline that is `cline_mcp_settings.json`.

With an API key:

```json
{
  "mcpServers": {
    "domainee": {
      "type": "streamableHttp",
      "url": "https://mcp.domainee.dev/mcp",
      "headers": {
        "Authorization": "Bearer sk_live_YOUR_KEY"
      }
    }
  }
}
```

Without a key (free diagnostic tools only), leave out `headers`.

## Verify the installation

Call `tools_ssl_check` with `host` set to `domainee.dev`. It needs no key and should return the
certificate details. With a key, also call `list_domains`, which returns the workspace's
domains (an empty list is fine on a new workspace).
