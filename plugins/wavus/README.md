# Wavus

Wavus integration — search for customers, companies, investors, and key
decision-makers in one place, with natural-language prospect search, fit scoring,
and data enrichment, via the remote Wavus MCP server.

- Product: <https://www.wavus.ai/overview>
- Endpoint: `https://mcp-dev.corepass.com/mcp` (Streamable HTTP) — **see the
  warning below**

## ⚠️ This points at a dev endpoint

`mcp-dev.corepass.com` is the **development** host. There is no production
endpoint yet: `mcp.corepass.com` does not complete a TLS handshake as of
2026-07-31.

That has a direct consequence for publishing. A catalog row for this entry sends
every installer to a dev server, so keep it out of the published catalog until a
prod host exists — insert it with `status = 'DRAFT'` rather than `'PUBLISHED'`
(the `uc_plugin` default is already `DRAFT`), or hold the row entirely. When the
prod endpoint lands, this is a one-line change to `.mcp.json` plus a `version`
bump; nothing else here is dev-specific.

## What this plugin is

One MCP server, nothing else — no skills, commands, agents or hooks:

```json
{
  "mcpServers": {
    "wavus": {
      "type": "http",
      "url": "https://mcp-dev.corepass.com/mcp"
    }
  }
}
```

Like `sfranalytics` and `notion`, this directory is **not vendored from an
upstream repo** — Wavus ships a hosted MCP endpoint, not a plugin repository. The
manifest and `.mcp.json` above are authored here, which is why the marketplace
entry carries no `_provenance`: there is no upstream commit to pin.

## What Wavus does

An AI intelligence platform for finding and reaching decision-makers — its own
framing is "search for customers, companies, investors, and key decision-makers
in one place". Four cooperating agents (discovery, planning, data processing,
prioritization) turn a conversational description of a target prospect into
verified contacts, ranked by a fit score, with enrichment on investment mandates,
AUM, funding history, contact details and deal activity. Aimed at capital
markets, real estate and lending, and B2B prospecting.

Categorized as `data-analytics`, matching `sfranalytics` and the wider
company/contact-intelligence group.

## Authentication

Browser-based OAuth at connect time, with **no credential to configure** — no API
key to paste, no client id, no client secret. The server advertises RFC 7591
dynamic client registration (`https://mcp-dev.corepass.com/register`), so the
client registers its own public PKCE client during the first authorization.

Verified against the live endpoint:

- `initialize` → `401` with
  `WWW-Authenticate: Bearer resource_metadata="…/.well-known/oauth-protected-resource/mcp"`
  (RFC 9728, correctly path-scoped)
- Protected-resource metadata advertises
  `scopes_supported: ["mcp:read", "offline_access"]` and
  `authorization_servers: ["https://mcp-dev.corepass.com"]`
- DCR `POST /register` → `201` with a `client_id`

Because the server advertises its scopes, the discovery fallback in
`client_engine` picks up `mcp:read offline_access` on its own and no `oauth.scopes`
needs declaring here. `offline_access` is what yields a refresh token, so
`refresh_client_token` can rotate it rather than forcing a re-authorization.

Notably this is the shape Datadog lacks — Datadog advertises no
`scopes_supported`, which is how the empty-`scope=` authorize bug surfaced. Wavus
does not hit that path.

## Why the config is just a URL

A connector that supports DCR has no static client id to declare — it mints one
per authorization — so there is nothing to put in an `oauth` block. There is no
`route: "local"` either: that key asserts the provider's DCR/authorize flow
accepts *only* a loopback `redirect_uri`, which has not been established here,
and it would be inert on the plugin path regardless
(`plugin_mcp.to_config_dict` drops it). `mcpConfigCanUseLocalOAuth` gates Connect
on a URL with no `headers` / `headersHelper`, and the 401 drives discovery from
there.

## Upgrading this entry

The endpoint is a live service, so there is no sha to bump — `version` tracks
*our* packaging, not the Wavus API. The change to expect is the dev → prod host
swap above.
