# Exa

Exa integration — neural web search, page-content retrieval, and deep research built for agents, via the remote Exa MCP server.

- Docs: <https://docs.exa.ai/reference/exa-mcp>
- Endpoint: `https://mcp.exa.ai/mcp` (Streamable HTTP)

## What this plugin is

One MCP server, nothing else — no skills, commands, agents or hooks:

```json
{
  "mcpServers": {
    "exa": {
      "type": "http",
      "url": "https://mcp.exa.ai/mcp"
    }
  }
}
```

Like `sfranalytics` and `notion`, this directory is **not vendored from an
upstream repo** — Exa ships a hosted MCP endpoint, not a plugin
repository. The manifest and `.mcp.json` above are authored here, which is why
the marketplace entry carries no `_provenance`: there is no upstream commit to
pin.

## Authentication

The endpoint answers `initialize` **anonymously**, so this plugin works with no
connect step. Exa also advertises DCR
(`https://auth.exa.ai/api/oauth/register`, scope `mcp:tools`) for accounts that
need higher limits — connecting is possible when the anonymous tier stops being
enough, and needs no config change to allow it.

## Upgrading this entry

The endpoint is a live service, so there is no sha to bump — `version` tracks
*our* packaging of the connector, not Exa's API. Bump it when the config
here changes (a new URL, an added server, an `auth` block), not when Exa
ships new tools.
