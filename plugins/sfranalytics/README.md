# SFR Analytics

Single-family-rental property intelligence: screen deals, research buyers, pull
rental comps, and rank markets. The vendor advertises 71+ tools across property
transactions, rental comps, buyer intelligence and market rankings.

- Product: <https://www.sfranalytics.com>
- Install page: <https://mcp.sfranalytics.com/install>

## What this plugin is

One MCP server, nothing else — no skills, commands, agents or hooks:

```json
{
  "mcpServers": {
    "sfranalytics": {
      "type": "http",
      "url": "https://mcp.sfranalytics.com/mcp"
    }
  }
}
```

Unlike the other plugins here, this directory is **not vendored from an upstream
repo** — SFR Analytics ships no plugin repository, only a hosted endpoint. The
manifest and `.mcp.json` above are authored here, which is why the marketplace
entry carries no `_provenance`: there is no upstream commit to pin.

## Authentication

Browser-based OAuth against SFR Analytics, handled at connect time — there is no
API key to paste and no client secret to configure, which is why no `auth` block
appears in `.mcp.json`. A hosted HTTP server with a literal URL and no
`${ENV}` header placeholders is what the client reads as "Authenticate"; the same
shape the Figma and Slack servers use.

The vendor offers 1,000 free credits without a card, so the connect flow can be
exercised end to end before any billing decision.

## Upgrading this entry

The endpoint is a live service, so there is no sha to bump — `version` in the
manifest tracks *our* packaging of it, not the vendor's API. Bump it when the
config here changes (a new URL, added scopes, an `auth` block), not when SFR
ships tools.
