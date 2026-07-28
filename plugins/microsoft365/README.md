# Microsoft 365 MCP Plugin

A [Corepass Code][corepass] / [Claude Code][claude-code] / [Cursor][cursor] plugin that
brings **Microsoft 365** into your AI tools — Outlook email, calendar, OneDrive,
SharePoint, and Teams — through the [Microsoft Graph API][graph].

It wraps the open-source [`@softeria/ms-365-mcp-server`][upstream] (MIT), run
locally over **stdio**. No hosted infrastructure and no custom Azure app
registration are required for the default flow — the server handles sign-in with
Microsoft's [device-code flow][device-code] and caches tokens in your OS
credential store.

## Installation

```text
/plugin install microsoft365@corepass-plugins
```

The MCP server is configured automatically (it runs via `npx`, so
[Node.js][node] ≥ 20 must be on your PATH). On first use, ask the assistant to
sign in — it will surface a URL and a one-time code. Open the URL, enter the
code, and approve the requested Microsoft Graph permissions. Subsequent runs
reuse the cached token.

## Features

The plugin exposes Microsoft 365 tools via the Graph API:

- **Mail** — search, read, send, reply to, update, and delete Outlook messages
- **Calendar** — list, create, and update events
- **Files** — browse and read OneDrive / SharePoint documents
- **Teams / Contacts** — read chats, channels, and contacts

## Usage examples

Once installed, talk to your tool in natural language:

- "Search my inbox for the invoice from Acme last week"
- "Summarize the unread emails in my Outlook inbox"
- "Draft a reply to the latest message from rico@youland.com"
- "What's on my calendar tomorrow?"
- "Find the Q3 planning doc in my OneDrive"

## Configuration

The default config runs the full Microsoft 365 tool set for a personal or
work/school mailbox:

```json
{
  "mcpServers": {
    "microsoft365": {
      "command": "npx",
      "args": ["-y", "@softeria/ms-365-mcp-server"]
    }
  }
}
```

Common adjustments (edit `.mcp.json` or your local `mcp.json`):

- **Email only** — add `"--preset", "mail"` to `args` to expose just the Outlook
  mail tools and keep the tool list lean.
- **Work / school with shared mailboxes** — add `"--org-mode"` to `args`
  (enables `Mail.Read.Shared` / `Mail.Send.Shared` scopes and admin-consent).
- **Your own Azure app** — set `MS365_MCP_CLIENT_ID` (and optionally
  `MS365_MCP_TENANT_ID`) in an `env` block to use a registered application
  instead of the default public client.
- **Bring your own token** — set `MS365_MCP_OAUTH_TOKEN` to inject an existing
  Graph access token and skip interactive login.

Inspect the exact Graph permissions a given config requests:

```bash
npx -y @softeria/ms-365-mcp-server --preset mail --list-permissions
```

## Requirements

- [Node.js][node] ≥ 20 on your PATH (the server is launched with `npx`).
- A Microsoft account. Personal accounts (`@outlook.com`, `@hotmail.com`) work
  out of the box; work/school accounts may require tenant admin consent for the
  requested Graph scopes.

## Attribution

This plugin is a thin wrapper around [`@softeria/ms-365-mcp-server`][upstream],
authored by Softeria and licensed under MIT. The plugin itself is not affiliated
with or endorsed by Microsoft.

[corepass]: https://corepass.com
[claude-code]: https://claude.com/claude-code
[cursor]: https://cursor.com
[graph]: https://learn.microsoft.com/en-us/graph/overview
[upstream]: https://github.com/softeria/ms-365-mcp-server
[device-code]: https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-device-code
[node]: https://nodejs.org
