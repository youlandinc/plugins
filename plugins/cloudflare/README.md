# Cloudflare

Cloudflare integration — search Cloudflare docs, manage Workers bindings (D1, KV, R2, Hyperdrive), inspect Workers builds, and query Workers logs and analytics.

- Docs: <https://developers.cloudflare.com/agents/model-context-protocol/mcp-servers-for-cloudflare/>
- Endpoints: 4 Streamable HTTP servers (see below)

## What this plugin is

4 MCP servers, nothing else — no skills, commands, agents or hooks:

```json
{
  "mcpServers": {
    "cloudflare-docs": {
      "type": "http",
      "url": "https://docs.mcp.cloudflare.com/mcp"
    },
    "cloudflare-bindings": {
      "type": "http",
      "url": "https://bindings.mcp.cloudflare.com/mcp"
    },
    "cloudflare-builds": {
      "type": "http",
      "url": "https://builds.mcp.cloudflare.com/mcp"
    },
    "cloudflare-observability": {
      "type": "http",
      "url": "https://observability.mcp.cloudflare.com/mcp"
    }
  }
}
```

Like `sfranalytics` and `notion`, this directory is **not vendored from an
upstream repo** — Cloudflare ships a hosted MCP endpoint, not a plugin
repository. The manifest and `.mcp.json` above are authored here, which is why
the marketplace entry carries no `_provenance`: there is no upstream commit to
pin.

## The four servers

Cloudflare splits its MCP surface by capability, and this plugin ships the four
that matter for app work. They are independent endpoints, not one server with
four modes:

| Server | Endpoint | Auth |
|---|---|---|
| `cloudflare-docs` | `docs.mcp.cloudflare.com/mcp` | none — answers `initialize` anonymously |
| `cloudflare-bindings` | `bindings.mcp.cloudflare.com/mcp` | OAuth (DCR) |
| `cloudflare-builds` | `builds.mcp.cloudflare.com/mcp` | OAuth (DCR) |
| `cloudflare-observability` | `observability.mcp.cloudflare.com/mcp` | OAuth (DCR) |

`cloudflare-docs` needs no connect step at all; the other three drive the
browser OAuth flow off their 401. Cloudflare runs more servers than these four
(Radar, Browser Rendering, AI Gateway, Container, …); add them as further
entries in the same `mcpServers` map when they are wanted.

## Authentication

Browser-based OAuth at connect time, with **no credential to configure** — no
API key to paste, no client id, no client secret. Cloudflare advertises RFC 7591
**dynamic client registration** (`https://bindings.mcp.cloudflare.com/register`), so the client
registers its own public PKCE client during the first authorization.

That is why `.mcp.json` is a bare URL — see below.

## Why the config is just a URL

A connector that supports DCR has no static client id to declare — it mints one
per authorization — so there is nothing to put in an `oauth` block. Writing
`"oauth": {"clientId": ""}` to satisfy a non-empty check would be a field that
will never hold a value.

There is no `route: "local"` either, unlike `linear` and `sfranalytics`. That
key asserts something specific about the provider: that its DCR/authorize flow
accepts *only* a loopback `redirect_uri`, so edge's hosted HTTPS callback can
never complete the handoff. That was established for sfranalytics; it has not
been established for this connector, and the key is not a generic "uses DCR"
marker. (It would also be inert here — `plugin_mcp.to_config_dict` drops
`route`, so it reaches neither `connector_needs_auth` nor the renderer's
`entryHasAuthBlock`.)

The connect path does not need either key. `mcpConfigCanUseLocalOAuth` gates the
Connect control on a URL with no `headers` / `headersHelper`, and the remote's
401 drives RFC 9728/8414 discovery and DCR from there — the same way `notion`
connects today.

## Upgrading this entry

The endpoint is a live service, so there is no sha to bump — `version` tracks
*our* packaging of the connector, not Cloudflare's API. Bump it when the config
here changes (a new URL, an added server, an `auth` block), not when Cloudflare
ships new tools.
