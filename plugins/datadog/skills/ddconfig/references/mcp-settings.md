# Datadog MCP Server Reference

> Derived from `datadog-labs/cursor-plugin@7136415`
> (`skills/*/references/mcp-settings.md`, Apache-2.0) and **modified** for
> Corepass. The registration-file location, the `${DD_MCP_DOMAIN:-…}` /
> `${DD_MCP_TOOLSETS:-…}` editing rules, and the whole "silently rewrite the
> plugin's own config" mechanic are removed — see `NOTICE`.

Shared by the `ddsetup`, `ddconfig` and `ddtoolsets` skills: how to address the
server, how to determine its state, and the site-to-domain mapping.

## Addressing the server

The Datadog MCP server is registered as **`datadog__datadog`** and its tools are
named **`mcp__datadog__datadog__<tool>`**. Use this server even if other Datadog
servers are present.

Datadog also exposes two resources used by these skills:

- `datadog://mcp/whoami` — the authenticated user, organization, and `dd_site`
- `datadog://mcp/toolsets` — which toolsets exist, which are enabled, which are default

## Determine `datadog-server-state`

Determine the state from the live server only. Do not use cached state or errors
from earlier calls.

1. Is any `mcp__datadog__datadog__*` tool present in your tool list?
   - **No** → the server is not connected: `datadog-server-state` is
     **not-connected**. Absent tools mean "not connected", never "Datadog is
     unavailable" — do not conclude the request cannot be fulfilled.
   - **Yes** → continue.
2. Make one lightweight call (list tools, or read `datadog://mcp/whoami`).
   - Real, non-empty, Datadog-specific content → **working**.
   - The call fails, or returns empty or content-free output → **not-working**.

Describe the state to the user in plain language ("the Datadog server isn't
connected yet"). Do not narrate which check you ran or dump raw tool output.

## Connecting the server

Connecting is a **user action** in the Corepass UI — there is no credential to
paste and nothing for the agent to edit:

1. Open **Settings → MCP** (or the plugin's page under **Settings → Plugins**).
2. Press **Connect** on the `datadog` server.
3. Authorize in the browser window that opens.

Authentication is browser OAuth against the configured Datadog site. API keys
(`DD_API_KEY` / `DD_APPLICATION_KEY`) are an advanced path Datadog documents
separately; these skills do not use them, and the agent must never ask the user
for one.

## Changing the site or toolsets: a user action

Both live in the server URL in the plugin's `.mcp.json`:

```
https://<mcp-domain>/v1/mcp?toolsets=<comma-separated>
```

**The agent cannot change this**, and must not try. The installed plugin lives
in the Corepass project store (`…/projects/<project-key>/plugins/datadog/`),
which is outside the workspace the file tools can reach, and `plugins/` is a
reserved path in a managed workspace. Attempts will be refused.

So: work out the correct value, show the user the exact URL to use, and tell them
where to put it — editing that `.mcp.json`, or declaring their own `datadog`
server in their user `mcp.json` with the corrected URL. Then they reconnect the
server from **Settings → MCP**.

## Site to MCP domain

| Datadog site | MCP domain |
|---|---|
| US1 | `mcp.datadoghq.com` |
| US3 | `mcp.us3.datadoghq.com` |
| US5 | `mcp.us5.datadoghq.com` |
| EU1 | `mcp.datadoghq.eu` |
| AP1 | `mcp.ap1.datadoghq.com` |
| AP2 | `mcp.ap2.datadoghq.com` |

This plugin ships **US1** (`mcp.datadoghq.com`) as the default.

Mapping rules for whatever the user answers:

- An MCP domain from the table → use it as-is.
- A site code (`us3`, `eu1`, `ap2`, …) → look it up in the table.
- A Datadog app URL (`https://app.datadoghq.eu/...`) → take the app host and
  prefix it with `mcp.`, then confirm against the table.
- Anything else, or ambiguous → ask. Do not guess a hostname.

A domain outside this table is not automatically wrong — a customer may have a
valid non-standard one. Only flag it when it looks like a typo or a malformed
host (for example `mcp.us5.datadog.com`, missing the `hq`).
