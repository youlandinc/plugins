# Asana

Asana integration — search and manage tasks, projects, portfolios, and goals, and post updates across your Asana workspace, via the remote Asana MCP server.

- Docs: <https://developers.asana.com/docs/using-asanas-mcp-server>
- Endpoint: `https://mcp.asana.com/mcp` (Streamable HTTP)

## What this plugin is

One MCP server, nothing else — no skills, commands, agents or hooks:

```json
{
  "mcpServers": {
    "asana": {
      "type": "http",
      "url": "https://mcp.asana.com/mcp"
    }
  }
}
```

Like `sfranalytics` and `notion`, this directory is **not vendored from an
upstream repo** — Asana ships a hosted MCP endpoint, not a plugin
repository. The manifest and `.mcp.json` above are authored here, which is why
the marketplace entry carries no `_provenance`: there is no upstream commit to
pin.

## Endpoint choice

Asana documents both an SSE endpoint (`/sse`) and a Streamable HTTP one
(`/mcp`). This plugin uses `/mcp`: both answer, but Streamable HTTP is the
current MCP transport and the one the desktop client prefers.

## Authentication

Browser-based OAuth at connect time, with **no credential to configure** — no
API key to paste, no client id, no client secret. Asana advertises RFC 7591
**dynamic client registration** (`https://mcp.asana.com/register`), so the client
registers its own public PKCE client during the first authorization, requesting `default`.

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
*our* packaging of the connector, not Asana's API. Bump it when the config
here changes (a new URL, an added server, an `auth` block), not when Asana
ships new tools.
