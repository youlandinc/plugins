# Context7

Context7 integration — pull version-accurate documentation and code examples for any library straight into the model's context, instead of relying on stale training data.

- Docs: <https://context7.com>
- Endpoint: `https://mcp.context7.com/mcp` (Streamable HTTP)

## What this plugin is

One MCP server, nothing else — no skills, commands, agents or hooks:

```json
{
  "mcpServers": {
    "context7": {
      "type": "http",
      "url": "https://mcp.context7.com/mcp"
    }
  }
}
```

Like `sfranalytics` and `notion`, this directory is **not vendored from an
upstream repo** — Context7 ships a hosted MCP endpoint, not a plugin
repository. The manifest and `.mcp.json` above are authored here, which is why
the marketplace entry carries no `_provenance`: there is no upstream commit to
pin.

## Authentication

The endpoint answers `initialize` **anonymously** — this plugin works with no
connect step at all, and the pre-connect probe learns that from the 200 rather
than from anything declared in config.

Context7 does advertise DCR (`https://context7.com/api/oauth/register`), and
signing in raises the anonymous rate limit. Until the storefront can express
"optional auth", the anonymous path is the honest default.

## Upgrading this entry

The endpoint is a live service, so there is no sha to bump — `version` tracks
*our* packaging of the connector, not Context7's API. Bump it when the config
here changes (a new URL, an added server, an `auth` block), not when Context7
ships new tools.
