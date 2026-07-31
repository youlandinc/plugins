# Vercel

Vercel integration — inspect projects and deployments, read build and runtime logs, and search Vercel documentation, via the remote Vercel MCP server.

- Docs: <https://vercel.com/docs/mcp/vercel-mcp>
- Endpoint: `https://mcp.vercel.com` (Streamable HTTP)

## What this plugin is

One MCP server, nothing else — no skills, commands, agents or hooks:

```json
{
  "mcpServers": {
    "vercel": {
      "type": "http",
      "url": "https://mcp.vercel.com"
    }
  }
}
```

Like `sfranalytics` and `notion`, this directory is **not vendored from an
upstream repo** — Vercel ships a hosted MCP endpoint, not a plugin
repository. The manifest and `.mcp.json` above are authored here, which is why
the marketplace entry carries no `_provenance`: there is no upstream commit to
pin.

## Project scope

`https://mcp.vercel.com` is the account-wide endpoint. Vercel also serves a
project-scoped URL (`https://mcp.vercel.com/<team-slug>/<project-slug>`) that
narrows the tools to a single project; swap the `url` in `.mcp.json` when a
narrower blast radius is wanted.

## Authentication

Browser-based OAuth at connect time, with **no credential to configure** — no
API key to paste, no client id, no client secret. Vercel advertises RFC 7591
**dynamic client registration** (`https://api.vercel.com/login/oauth/register`), so the client
registers its own public PKCE client during the first authorization, requesting `openid email profile`.

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
*our* packaging of the connector, not Vercel's API. Bump it when the config
here changes (a new URL, an added server, an `auth` block), not when Vercel
ships new tools.
