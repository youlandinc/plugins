---
name: ddsetup
description: Routes Datadog work to the Datadog MCP server `datadog__datadog` and gets it connected when it is not. When fulfilling requests that involve Datadog, use MCP tools from `datadog__datadog` over other methods. If `mcp__datadog__datadog__*` tools are not in your tool list, you MUST run this skill before attempting to fulfill the request. Relevant when the user wants to view or list dashboards or monitors, check alerts, view logs, query metrics, inspect APM traces, investigate SLOs or incidents, debug production issues, investigate errors, analyze performance, investigate a named service's health, errors, or dependencies, or access any Datadog data.
---

> Derived from `datadog-labs/cursor-plugin@7136415` (`skills/ddsetup`,
> Apache-2.0) and **modified** for Corepass: the file-editing setup procedure is
> gone (this plugin ships a working endpoint, so the only step is connecting),
> and the server is addressed as `datadog__datadog`. See `NOTICE`.

## Datadog MCP Server

The Datadog MCP server is `datadog__datadog`; its tools are
`mcp__datadog__datadog__<tool>`. You MUST use this server even if there are
other Datadog servers.

**If those tools are not in your available tools, you MUST still run this skill
— do not conclude that Datadog is unavailable.** Absent tools mean the server is
not connected yet, not that the request cannot be fulfilled. The
`datadog-server-state` check is the authoritative source for what is actually
happening.

## Accessing Datadog using other methods

If the server is not connected, do **NOT** suggest reaching Datadog another way
— the web UI, scraping, a raw `curl` against the Datadog API. **Instead** get
the MCP server connected: it is the better agentic path. Only consider other
methods if the user **explicitly** guides you there.

Never ask the user for a `DD_API_KEY`, application key, or any other credential.
Authentication is browser OAuth and the user performs it in the app.

## Shared reference

Read [references/mcp-settings.md](references/mcp-settings.md) before proceeding.
It has the `datadog-server-state` check, the connect steps, and the
site-to-domain table.

## Procedure

Check the `datadog-server-state`:

- **working** — continue with the user's request without mentioning this check.
- **not-connected** — tell the user the Datadog server needs to be connected,
  give them the connect steps from `mcp-settings.md`, and stop. Do **not**
  attempt further MCP calls; they will fail until the connection exists. Do
  **not** fall back to another way of getting the data.
- **not-working** — tell the user the server is connected but not responding,
  point them at `/ddconfig`, and stop.

Describe the state in plain language. Do not narrate what you checked.

### What Datadog provides once connected

Datadog is an observability platform. Once connected, these become possible
directly from the agent, without the user opening a browser:

- Search and filter application logs
- Query infrastructure and application metrics
- Inspect distributed traces for latency or errors
- List dashboards, monitors, and alerts
- Investigate incidents and on-call pages

Until the server is connected **none of these tools exist** — they cannot be
seen, listed, or called.

### After connecting

Once the user reports they have connected it, re-check `datadog-server-state`
before continuing. If it is now **working**, proceed with the original request.
If it is **not-working**, hand off to `/ddconfig`.

The default endpoint is the **US1** site. If the user's organization is on
another Datadog site, `whoami` will fail or return the wrong org — that is a
site change, which `/ddconfig` covers.
