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

The server exposes two tiers. The **16 free diagnostic tools are always
available with no auth**; the **11 workspace tools require a Bearer API key** and
are only registered when a token is present. An unauthenticated `tools/list`
returns the 16 free tools; add a key to get all 27.

**Workspace tools (11, require a Bearer API key)**

- Domains — `list_domains`, `get_domain`, `create_domain`, `update_domain`,
  `delete_domain`, `check_domain` (force an immediate DNS/SSL probe)
- Webhook endpoints — `list_webhook_endpoints`, `create_webhook_endpoint`,
  `delete_webhook_endpoint`
- DNS checks — `dns_check_records_exist`, `dns_check_records_match_exactly`

**Free diagnostic tools (16, no key)**

Read-only SSL/DNS/WHOIS lookups: `tools_ssl_check`, `tools_dns_record_lookup`,
`tools_whois_lookup`, `tools_cname_lookup`, `tools_http_header_checker`,
`tools_dns_propagation_checker`, `tools_redirect_checker`, `tools_spf_record_checker`,
`tools_dkim_record_checker`, `tools_dmarc_record_checker`, `tools_txt_record_lookup`,
`tools_domain_age_checker`, `tools_domain_availability_checker`, `tools_subdomain_finder`,
`tools_reverse_ip_lookup`, `tools_website_status_checker`.

## Pricing

Free tier: 50 custom domains + 100 GB bandwidth/month, no credit card. Usage pricing
beyond that at <https://domainee.dev/pricing>.

## Links

- Product: <https://domainee.dev>
- MCP docs: <https://domainee.dev/mcp> · <https://domainee.dev/docs/mcp>
- API docs: <https://domainee.dev/docs>
- Registry: `dev.domainee/domainee` on <https://registry.modelcontextprotocol.io>
