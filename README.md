# Domainee MCP Server

> Custom domains API for SaaS, exposed as Model Context Protocol tools. Connect,
> verify, and manage **your customers'** domains, SSL, and DNS from any MCP client.
> 20 domains free.

[![MCP Registry](https://img.shields.io/badge/MCP_Registry-dev.domainee%2Fdomainee-blue)](https://registry.modelcontextprotocol.io)
[![Glama](https://img.shields.io/badge/Glama-healthy-green)](https://glama.ai/mcp/connectors/dev.domainee/domainee)
[![Smithery](https://img.shields.io/badge/Smithery-80%2F100-green)](https://smithery.ai/server/admin-cl6n/domainee)
[![Claude Directory](https://img.shields.io/badge/Claude_Directory-community-orange)](https://domainee.dev/mcp)
[![License](https://img.shields.io/badge/license-MIT-lightgrey)](LICENSE)

Domainee is a custom domains API for SaaS with a native MCP server, 20 domains and
100 GB free. This is the hosted, stateless [Model Context Protocol](https://modelcontextprotocol.io)
server that lets AI agents (Claude, Cursor, Windsurf, and any MCP-aware client) drive
the Domainee API directly: onboard a customer's hostname, hand them DNS steps written
for their actual DNS provider (or a one-click approval link when their DNS is on
Cloudflare), force DNS and SSL probes, debug "my domain isn't working" tickets, and manage webhook
endpoints. Same Bearer key, rate limits, and workspace as the REST API. Nothing to deploy.

## This is not a registrar MCP server

Most domain MCP servers wrap a registrar's API so an agent can manage **domains you
own** — search, buy, renew, edit nameservers at GoDaddy / Namecheap / Dynadot / NameSilo.

Domainee solves the other problem: letting **your customers point their own domains at
your app**. You're a SaaS, your customer owns `shop.acme.com`, and you need it serving
their storefront over HTTPS in a minute without them touching your infrastructure. That
means custom-hostname routing, automatic certificate issuance and renewal, DNS
verification, and a webhook when it goes live — the "Cloudflare for SaaS" shape, not the
registrar shape.

If you want an agent to buy you a domain, use a registrar server. If you want an agent
to onboard a customer's domain into your product, this is the one.

## Endpoint

```
https://mcp.domainee.dev/mcp
```

Remote, streamable HTTP, stateless. No install, no local process, nothing to deploy.

**Auth is optional for discovery.** `initialize`, `tools/list`, and all 18 free
diagnostic tools answer anonymously. A Domainee API key (`sk_live_…`, minted at
<https://domainee.dev/developers>) unlocks the 12 workspace tools; calling one without a
key returns `401` plus a `WWW-Authenticate` header pointing at our OAuth metadata, which
is what starts the OAuth flow in MCP clients. OAuth is also supported end to end
(RFC 9728 protected-resource metadata + RFC 8414 authorization-server metadata).

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

Try it with no key at all — this returns the 18 free tools as a `text/event-stream`
response:

```bash
curl -s https://mcp.domainee.dev/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' \
  | grep '^data: ' | sed 's/^data: //' | jq '.result.tools | length'
# 18
```

## Tools

30 tools in two tiers. The **18 free diagnostic tools need no key**; the **12 workspace
tools require a Bearer** and are only registered when a token is present, so an
unauthenticated `tools/list` returns 18 and an authenticated one returns 30.

### Workspace tools (12, Bearer API key required)

| Tool | What it does | Key arguments |
|---|---|---|
| `list_domains` | List custom domains in the workspace. Cursor-paginated. | `status` (`pending`/`verified`/`failed`/`expired`), `hostname` (exact match), `limit`, `cursor` |
| `get_domain` | Full Domain object: hostname, originUrl, status, monitorStatus, dnsRecords. | `id` |
| `create_domain` | Register a customer hostname for proxy or redirect through the edge. Returns preflight warnings (CAA, unreachable origin). | `hostname`, `originUrl`, `mode` (`proxy`/`redirect`), `keepHost`, `redirectWww`, `redirectStatus`, `metadata` |
| `update_domain` | Edit a domain. Hostname is immutable — delete and recreate to change it. | `id` + any of `originUrl`, `mode`, `keepHost`, `redirectWww`, `redirectStatus`, `metadata` |
| `delete_domain` | Stop routing a hostname; the edge stops serving within ~60s. | `id` |
| `get_connect_instructions` | DNS setup steps for the provider actually serving the customer's domain: exact record type and name, where the record editor is, provider gotchas (Cloudflare proxy, GoDaddy parked A record). Also returns `domainConnect`: a one-click link where the DNS provider has enabled Domainee's Domain Connect templates (live at Cloudflare for all accounts, and Glauca Digital), otherwise a `reason`. Call `check_domain` once the customer approves. | `id` |
| `check_domain` | Force an immediate DNS/SSL probe instead of waiting for the next monitor tick. Flips `pending` → `verified`. | `id` |
| `list_webhook_endpoints` | List webhook endpoints. Signing secrets are never included — they are revealed once, at create time. | — |
| `create_webhook_endpoint` | Register an HTTPS URL that Domainee POSTs domain events to. Returns the signing secret once. | `url`, `events[]` (empty = all) |
| `delete_webhook_endpoint` | Stop sending events; in-flight retries are dropped. | `id` |
| `dns_check_records_exist` | Per record, `match: true` if **at least one** DNS value equals the expected one. Tolerates extra records alongside the CNAME. Up to 50 per call. | `records[]` of `{ address, type, match_against }` |
| `dns_check_records_match_exactly` | Stricter: `match: true` only if **every** returned value matches. Use when the customer must point only at you. | `records[]` |

`type` accepts `a`, `aaaa`, `cname`, `mx`, `txt`, `ns`, `caa`. Comparison is
case-insensitive and trailing dots are stripped. Webhook events are
`domain.created`, `domain.verified`, `domain.failed`, `domain.expired`,
`domain.deleted`, `domain.monitor_updated`.

### Free diagnostic tools (18, no key, no signup)

Read-only SSL / DNS / WHOIS lookups, usable by any agent against any domain:

`tools_ssl_check`, `tools_dns_provider_lookup`, `tools_domain_connect_checker`,
`tools_dns_record_lookup`, `tools_whois_lookup`,
`tools_cname_lookup`, `tools_http_header_checker`, `tools_dns_propagation_checker`,
`tools_redirect_checker`, `tools_spf_record_checker`, `tools_dkim_record_checker`,
`tools_dmarc_record_checker`, `tools_txt_record_lookup`, `tools_domain_age_checker`,
`tools_domain_availability_checker`, `tools_subdomain_finder`,
`tools_reverse_ip_lookup`, `tools_website_status_checker`.

The same 18 are available as keyless REST endpoints at
`https://api.domainee.dev/v1/tools/<name>` — see <https://domainee.dev/free-apis>.

## What agents do with it

**Onboard a customer domain.** "Connect `shop.acme.com` to `https://acme.myapp.com`."
The agent calls `create_domain`, then `get_connect_instructions` to tell the customer
exactly what to publish, in the words their DNS provider's panel uses. If their DNS is on
Cloudflare, it can hand them a one-click link instead: they approve the records on
Cloudflare's own screen and type nothing. Once they're done, it calls `check_domain` to
verify immediately rather than waiting for the monitor.

**Debug a broken domain.** "Why is `shop.acme.com` showing a certificate error?" The
agent calls `get_domain` for our view of the status, then `tools_cname_lookup` and
`tools_ssl_check` for ground truth from public DNS, and can tell the difference between
"the customer never published the record", "it's published but not propagated yet", and
"there's a CAA record blocking issuance".

**Audit the fleet.** "Which customer domains are failing?" — `list_domains` with
`status: "failed"`, then `dns_check_records_exist` across the results in one batched
call of up to 50 records.

## Pricing

Free tier: 20 custom domains + 100 GB bandwidth/month, with no expiry. You add a card
when you connect your first domain; it's charged $0 inside the free tier. Usage pricing
beyond that at <https://domainee.dev/pricing>. The 18 diagnostic tools are free and
keyless regardless.

## Listings

- MCP Registry — `dev.domainee/domainee` on <https://registry.modelcontextprotocol.io>
- Glama — <https://glama.ai/mcp/connectors/dev.domainee/domainee>
- Smithery — <https://smithery.ai/server/admin-cl6n/domainee>
- Claude connector directory — listed as a community connector; find it in Claude under
  Settings → Connectors

## Links

- Product: <https://domainee.dev>
- MCP docs: <https://domainee.dev/mcp> · <https://domainee.dev/docs/mcp>
- API docs: <https://domainee.dev/docs>
- Free keyless APIs: <https://domainee.dev/free-apis>
- For agents: <https://domainee.dev/llms.txt> · <https://domainee.dev/llms-full.txt>

## About this repo

Discovery and documentation only — the server is hosted, so there is nothing to clone
or run. It holds the `server.json` that the MCP Registry reads and this README. The
implementation lives in the Domainee monorepo.

Issues and questions are welcome here.

## License

MIT — see [LICENSE](LICENSE).
