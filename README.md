# Domainee MCP Server

> Custom domains API for SaaS, exposed as Model Context Protocol tools. Connect,
> verify, and manage customer domains, SSL, and DNS from any MCP client. 50 domains free.

[![MCP Registry](https://img.shields.io/badge/MCP_Registry-dev.domainee%2Fdomainee-blue)](https://registry.modelcontextprotocol.io)
[![Glama](https://img.shields.io/badge/Glama-listed-green)](https://glama.ai/mcp/connectors/dev.domainee/domainee)

Domainee is a custom domains API for SaaS with a native MCP server, 50 domains and
100 GB free. This is the hosted, stateless [Model Context Protocol](https://modelcontextprotocol.io)
server that lets AI agents (Claude, Cursor, Windsurf, and any MCP-aware client) drive
the Domainee API directly: onboard a customer's hostname and get back the CNAME target,
force DNS and SSL probes, debug "my domain isn't working" tickets, and manage webhook
endpoints. Same Bearer key, rate limits, and workspace as the REST API. Nothing to deploy.

## Endpoint

```
https://mcp.domainee.dev/mcp
```

Remote, streamable HTTP, stateless. Auth is a Domainee API key (`sk_live_…`) minted at
<https://domainee.dev/developers>.

## Install

Claude Code / CLI:

```bash
claude mcp add --transport http domainee https://mcp.domainee.dev/mcp \
  --header "Authorization: Bearer sk_live_YOUR_KEY"
```

Claude Desktop / Cursor / Windsurf (any HTTP-transport client):

```json
{
  "mcpServers": {
    "domainee": {
      "url": "https://mcp.domainee.dev/mcp",
      "headers": { "Authorization": "Bearer sk_live_YOUR_KEY" }
    }
  }
}
```

## Tools

**Domains** — `list_domains`, `get_domain`, `create_domain`, `update_domain`,
`delete_domain`, `check_domain` (force an immediate DNS/SSL probe)

**Webhook endpoints** — `list_webhook_endpoints`, `create_webhook_endpoint`,
`delete_webhook_endpoint`

**DNS checks** — `dns_check_records_exist`, `dns_check_records_match_exactly`

## Pricing

Free tier: 50 custom domains + 100 GB bandwidth/month, no credit card. Usage pricing
beyond that at <https://domainee.dev/pricing>.

## Links

- Product: <https://domainee.dev>
- MCP docs: <https://domainee.dev/mcp> · <https://domainee.dev/docs/mcp>
- API docs: <https://domainee.dev/docs>
- Registry: `dev.domainee/domainee` on <https://registry.modelcontextprotocol.io>
