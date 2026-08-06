# Wavus

Wavus integration — search for customers, companies, investors, and key
decision-makers in one place, with natural-language prospect search, fit scoring,
and data enrichment, via the remote Wavus MCP server.

- Product: <https://www.wavus.ai/overview>
- Endpoint: `https://mcp.corepass.com/mcp` (Streamable HTTP)

## What this plugin is

One MCP server, nothing else — no skills, commands, agents or hooks:

```json
{
  "mcpServers": {
    "wavus": {
      "type": "http",
      "url": "https://mcp.corepass.com/mcp"
    }
  }
}
```

Like `sfranalytics` and `notion`, this directory is **not vendored from an
upstream repo** — Wavus ships a hosted MCP endpoint, not a plugin repository. The
manifest and `.mcp.json` above are authored here, which is why the marketplace
entry carries no `_provenance`: there is no upstream commit to pin.

`logo.svg` is the one exception, and follows the same rule `sfranalytics` does:
fetched as a single asset from <https://www.wavus.ai/favicon.svg> (the only logo
the site exposes — there is no separate wordmark), not part of any vendored repo
or commit, so it gets no `_provenance` entry either. Re-fetch it by hand if Wavus
changes their branding. It is served to the storefront from this repo
(`raw.githubusercontent.com/youlandinc/plugins/main/plugins/wavus/logo.svg`)
rather than hotlinked, matching every other logo in the marketplace index.

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
dynamic client registration (`https://mcp.corepass.com/register`), so the client
registers its own public PKCE client during the first authorization.

Verified against the live production endpoint (2026-07-31):

- TLS 1.3 on both resolved ALB addresses, wildcard certificate for
  `*.corepass.com`
- `initialize` → `401` with
  `WWW-Authenticate: Bearer resource_metadata="https://mcp.corepass.com/.well-known/oauth-protected-resource/mcp"`
  (RFC 9728, correctly path-scoped rather than origin-wide)
- That metadata advertises
  `scopes_supported: ["mcp:read", "mcp:write", "offline_access"]` and
  `authorization_servers: ["https://mcp.corepass.com"]`
- DCR `POST /register` → `201` with a `client_id`

Because the server advertises its scopes, the discovery fallback in
`client_engine` resolves them on its own and no `oauth.scopes` needs declaring
here. `offline_access` is what yields a refresh token, so `refresh_client_token`
can rotate it rather than forcing a re-authorization.

(The advertised set gained `mcp:write` between two probes on 2026-07-31, so the
surface is still moving — worth re-reading rather than trusting this list.)

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
*our* packaging, not the Wavus API. Bump it when the config here changes (a new
URL, an added server, an `auth` block), not when Wavus ships new tools. The
advertised scope set is the thing most likely to move next.
