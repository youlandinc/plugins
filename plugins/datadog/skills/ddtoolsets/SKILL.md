---
name: ddtoolsets
description: Manages toolsets for the Datadog MCP server `datadog__datadog`. Use when the user wants to view, enable, or disable the toolsets that control which Datadog tools are available.
---

> Derived from `datadog-labs/cursor-plugin@7136415` (`skills/ddtoolsets`,
> Apache-2.0) and **modified** for Corepass: toolsets live in the server URL's
> `toolsets` query parameter rather than a `${DD_MCP_TOOLSETS:-…}` template, and
> the agent hands the change to the user instead of editing the file. See
> `NOTICE`.

## Datadog MCP Server

The Datadog MCP server is `datadog__datadog`; its tools are
`mcp__datadog__datadog__<tool>`. You MUST use this server even if there are
other Datadog servers.

## Shared reference

Read [references/mcp-settings.md](references/mcp-settings.md) before proceeding.

## Entry flow

Check the `datadog-server-state` and read the `datadog://mcp/toolsets` resource
from `datadog__datadog`. Output nothing until both are available, then:

- **working** and valid content — go to the [Toolsets flow](#toolsets-flow).
- **not-connected** — tell the user the server is not connected, point them at
  `/ddsetup`, and stop.
- **not-working** or invalid content — tell the user the server is configured
  but not responding, point them at `/ddconfig`, and stop.

## Toolsets flow

A toolset is a named group of tools for one Datadog feature. Enabling a toolset
makes its tools available; disabling it removes them.

### How toolsets are configured here

They are the `toolsets` query parameter on the server URL:

```
https://mcp.datadoghq.com/v1/mcp?toolsets=core,visualizations
```

Two states:

- **Absent** (no `toolsets=` at all) — the server picks. Preferred, because new
  server-side default toolsets are picked up automatically.
- **Explicit** — exactly the listed toolsets, nothing else. Server defaults are
  ignored, and a toolset Datadog adds later will NOT appear.

This plugin ships `toolsets=core,visualizations`, which mirrors upstream's
default.

Order is not meaningful — `core,alerting` and `alerting,core` are the same. When
comparing a computed list against the defaults, compare as sets, not strings.
Prefer dropping the parameter entirely over an explicit list that merely happens
to equal the current defaults.

### 1. Gather toolset information

Use the `datadog://mcp/toolsets` resource: which toolsets exist, which are
enabled, which are defaults, what each does. Present **all** of them — do not
summarize — in whatever format reads best (table, grouped list). Make it obvious
which are currently on.

Also read the current `toolsets` value from the server URL, visible in the
plugin's MCP configuration. Ignore any name there that the resource does not
list; drop it silently from any list you propose.

### 2. Understand the intent

The user may want to **add** to the enabled set, **remove** from it, or
**replace** it wholesale. Ask when ambiguous.

If no `toolsets` parameter is currently set and the user wants to add one, you
need to know what the defaults ARE to build the full list — take them from the
`datadog://mcp/toolsets` resource.

### 3. Compute the new value

- Result equals the default set exactly → drop the parameter.
- User asks to reset or use defaults → drop the parameter.
- Every toolset removed → drop the parameter, and warn that server defaults
  apply instead.
- Result omits `core` → warn before proceeding. `core` carries the essential
  functionality and most workflows depend on it. Only continue if the user
  explicitly confirms.
- Otherwise → an explicit comma-separated list.

### 4. Hand the change to the user

You cannot edit the plugin's `.mcp.json`; see `mcp-settings.md` for why. Give
the user the complete URL, keeping the current host:

```
https://<current-mcp-domain>/v1/mcp?toolsets=<new,list>
```

or, when dropping the parameter:

```
https://<current-mcp-domain>/v1/mcp
```

Tell them to set it as the `datadog` server's `url` — in the installed plugin's
`.mcp.json`, or in their own `mcp.json` — then reconnect the server from
**Settings → MCP**.

### 5. Confirm

Once they report it done, re-read `datadog://mcp/toolsets` and tell them which
toolsets are now enabled. If the set is unchanged, the edit did not take effect
— check the URL they actually applied.
