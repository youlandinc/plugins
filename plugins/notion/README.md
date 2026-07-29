# Notion

Connect AI assistants to your Notion workspace — search, read, create, and
update pages, databases, and comments through the remote Notion MCP server.

- Docs: <https://developers.notion.com/guides/mcp>
- Endpoint: `https://mcp.notion.com/mcp` (Streamable HTTP)

## What this plugin is

One MCP server, nothing else — no skills, commands, agents or hooks:

```json
{
  "mcpServers": {
    "notion": {
      "type": "http",
      "url": "https://mcp.notion.com/mcp"
    }
  }
}
```

Like `sfranalytics`, this directory is **not vendored from an upstream repo** —
Notion ships no Claude/Cursor plugin repository, only a hosted MCP endpoint. The
manifest and `.mcp.json` above are authored here, which is why the marketplace
entry carries no `_provenance`: there is no upstream commit to pin. `version`
tracks *our* packaging of the connector, not Notion's API.

`logo.svg` is the official Notion mark (paths from the CC0 simple-icons set).

## Authentication

Notion MCP requires **user-based OAuth** and does not support bearer-token
authentication, so the flow runs in the browser at connect time — no `auth`
block or `${ENV}` header placeholders, and no API key to paste.

## Upgrading this entry

The endpoint is a live service, so there is no sha to bump. Bump `version` when
the config here changes (a new URL, added scopes, an `auth` block), not when
Notion ships new tools.
