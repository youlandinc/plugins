# Stripe

Stripe integration — read and manage customers, products, prices, invoices, subscriptions, and disputes, and search Stripe documentation, via the remote Stripe MCP server.

- Docs: <https://docs.stripe.com/mcp>
- Endpoint: `https://mcp.stripe.com` (Streamable HTTP)

## What this plugin is

One MCP server, nothing else — no skills, commands, agents or hooks:

```json
{
  "mcpServers": {
    "stripe": {
      "type": "http",
      "url": "https://mcp.stripe.com"
    }
  }
}
```

Like `sfranalytics` and `notion`, this directory is **not vendored from an
upstream repo** — Stripe ships a hosted MCP endpoint, not a plugin
repository. The manifest and `.mcp.json` above are authored here, which is why
the marketplace entry carries no `_provenance`: there is no upstream commit to
pin.

## Live vs test data

The remote server authorizes against a Stripe **account**, and the tools act on
whatever mode that authorization grants. Treat it as production access: the read
tools are safe, but the write surface (customers, invoices, subscriptions,
refunds) is real money in a live account. There is no separate test-mode
endpoint to point at.

## Authentication

Browser-based OAuth at connect time, with **no credential to configure** — no
API key to paste, no client id, no client secret. Stripe advertises RFC 7591
**dynamic client registration** (`https://access.stripe.com/mcp/oauth2/register`), so the client
registers its own public PKCE client during the first authorization, requesting `mcp`.

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
*our* packaging of the connector, not Stripe's API. Bump it when the config
here changes (a new URL, an added server, an `auth` block), not when Stripe
ships new tools.
