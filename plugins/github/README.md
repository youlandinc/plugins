# GitHub

The official remote GitHub MCP server, vendored for the Corepass plugin
marketplace. Connect AI assistants to GitHub — manage repositories, issues,
pull requests, and workflows through natural language. No local install.

## What's included

- **MCP server** (`.mcp.json`) — the cloud-hosted GitHub MCP server at
  `https://api.githubcopilot.com/mcp/` (`streamable-http`). Authorize with your
  GitHub account via OAuth on first use.

## Authentication

- **OAuth** — on supported hosts, the server prompts you to sign in with GitHub
  on first use. Nothing to configure.
- **Personal Access Token (PAT)** — alternatively, pass a token via the
  `Authorization: Bearer <token>` header. Add it to the server config:

  ```json
  {
    "mcpServers": {
      "github": {
        "type": "http",
        "url": "https://api.githubcopilot.com/mcp/",
        "headers": { "Authorization": "Bearer ${GITHUB_PAT}" }
      }
    }
  }
  ```

## Toolset scoping (optional)

The remote server supports scoping via URL path or headers:

- `https://api.githubcopilot.com/mcp/readonly` — read-only mode
- `https://api.githubcopilot.com/mcp/x/issues` — a single toolset (e.g. issues, repos, actions)
- `https://api.githubcopilot.com/mcp/x/issues/readonly` — combine both
- or send headers `X-MCP-Toolsets` / `X-MCP-Readonly`

## Installation

```bash
harness marketplace add youlandinc/plugins
harness plugin install github
```

## License & provenance

- **Upstream**: <https://github.com/github/github-mcp-server>
- **License**: MIT (see `LICENSE`)

Only the remote connection config is vendored here — the upstream repository is
a full Go MCP server (also runnable locally / via Docker). All content remains
the copyright of GitHub.

## Support

- **GitHub MCP docs**: <https://github.com/github/github-mcp-server/blob/main/docs/remote-server.md>
- **Report issues**: <https://github.com/github/github-mcp-server/issues>
