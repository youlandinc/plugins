# Gmail

Connect AI assistants to your Gmail account — search, read, and compose email
through the remote Gmail MCP server.

- Docs: <https://developers.google.com/workspace/gmail/api/guides/configure-mcp-server>
- Endpoint: `https://gmailmcp.googleapis.com/mcp/v1` (HTTP)

## What this plugin is

One MCP server, nothing else — no skills, commands, agents or hooks:

```json
{
  "mcpServers": {
    "gmail": {
      "type": "http",
      "url": "https://gmailmcp.googleapis.com/mcp/v1",
      "oauth": {
        "clientId": "752782911539-8c82le89qbla3me0o1u5kcretpbtjf41.apps.googleusercontent.com",
        "callbackPort": 3118
      }
    }
  }
}
```

Like `notion` and `sfranalytics`, this directory is **not vendored from an
upstream repo** — Google ships a hosted MCP endpoint, not a plugin repository.
The manifest and `.mcp.json` above are authored here, which is why the
marketplace entry carries no `_provenance`. `version` tracks *our* packaging of
the connector, not the Gmail API.

`logo.png` is the official colorful Gmail mark, fetched as a single asset from
Google's branding CDN
(`https://www.gstatic.com/images/branding/product/2x/gmail_2020q4_96dp.png`).
It is a Google trademark, used here only to identify the connector.

## Authentication

The Gmail MCP server uses **OAuth 2.0**. The `oauth` block in `.mcp.json`
carries a public OAuth client ID (type *Web application*) and the local
`callbackPort`, so the browser authorization flow runs at connect time with no
secret to paste. The client secret is never stored in this manifest.

Typical scopes requested:

- `https://www.googleapis.com/auth/gmail.readonly`
- `https://www.googleapis.com/auth/gmail.compose`

The redirect URI is client-specific (for example
`https://claude.ai/api/mcp/auth_callback`). See
[Configure the Gmail MCP server](https://developers.google.com/workspace/gmail/api/guides/configure-mcp-server)
and [Auth overview](https://developers.google.com/workspace/guides/auth-overview)
for the full setup.

## Upgrading this entry

The endpoint is a live service, so there is no sha to bump. Bump `version` when
the config here changes (a new URL, added scopes, an `oauth` block), not when
Google ships new tools.
