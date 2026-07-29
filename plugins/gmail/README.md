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
      "url": "https://gmailmcp.googleapis.com/mcp/v1"
    }
  }
}
```

Like `notion` and `sfranalytics`, this directory is **not vendored from an
upstream repo** — Google ships a hosted MCP endpoint, not a plugin repository.
The manifest and `.mcp.json` above are authored here, which is why the
marketplace entry carries no `_provenance`. `version` tracks *our* packaging of
the connector, not the Gmail API.

`logo.svg` is the Gmail mark (path from the CC0 simple-icons set).

## Authentication

The Gmail MCP server uses **OAuth 2.0**, and unlike a fully hosted OAuth
connector it requires **your own OAuth 2.0 client** — you must create an OAuth
client ID and secret of type *Web application* in a Google Cloud project and
supply them to the client at connect time. There is no public client baked into
this manifest, so no `oauth` block appears in `.mcp.json`.

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
