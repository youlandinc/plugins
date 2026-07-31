# Sentry

Sentry integration — query issues, errors, traces, and releases across your Sentry organization, and run Seer root-cause analysis, via the remote Sentry MCP server.

- Docs: <https://docs.sentry.io/product/sentry-mcp/>
- Endpoint: `https://mcp.sentry.dev/mcp` (Streamable HTTP)

## What this plugin is

One MCP server, nothing else — no skills, commands, agents or hooks:

```json
{
  "mcpServers": {
    "sentry": {
      "type": "http",
      "url": "https://mcp.sentry.dev/mcp"
    }
  }
}
```

Like `sfranalytics` and `notion`, this directory is **not vendored from an
upstream repo** — Sentry ships a hosted MCP endpoint, not a plugin
repository. The manifest and `.mcp.json` above are authored here, which is why
the marketplace entry carries no `_provenance`: there is no upstream commit to
pin.

## Region

`mcp.sentry.dev` covers Sentry SaaS. Self-hosted Sentry and EU-region orgs are
reached through their own host — change the `url` rather than assuming one
endpoint serves every install.

## Authentication

Browser-based OAuth at connect time, with **no credential to configure** — no
API key to paste, no client id, no client secret. Sentry advertises RFC 7591
**dynamic client registration** (`https://mcp.sentry.dev/oauth/register`), so the client
registers its own public PKCE client during the first authorization, requesting `org:read project:write team:write event:write`.

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
*our* packaging of the connector, not Sentry's API. Bump it when the config
here changes (a new URL, an added server, an `auth` block), not when Sentry
ships new tools.
