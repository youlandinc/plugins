---
name: ddconfig
description: Configures or troubleshoots the Datadog MCP server `datadog__datadog`. Use when the user wants to change the Datadog site or MCP domain, switch organizations, or when the server is connected but not responding.
---

> Derived from `datadog-labs/cursor-plugin@7136415` (`skills/ddconfig`,
> Apache-2.0) and **modified** for Corepass: the agent no longer edits the
> plugin's registration file — it cannot reach it — so the domain flow produces
> an instruction for the user instead. Cursor UI steps are replaced. See
> `NOTICE`.

## Datadog MCP Server

The Datadog MCP server is `datadog__datadog`; its tools are
`mcp__datadog__datadog__<tool>`. You MUST use this server even if there are
other Datadog servers.

## Shared reference

Read [references/mcp-settings.md](references/mcp-settings.md) before proceeding.
It has the `datadog-server-state` check, the site-to-domain table, and why site
changes are a user action.

## Entry flow

Check the `datadog-server-state` and read the `datadog://mcp/whoami` resource
from `datadog__datadog`. Output nothing until both are available, then:

- **working** and valid content — show the user their current connection from
  `whoami`: user name and email, organization name, and site (`dd_site`). Then
  offer [changing the site or MCP domain](#domain-flow) or
  [switching organization](#organization-flow).
- **not-connected** — tell the user the server is not connected, point them at
  `/ddsetup`, and stop.
- **not-working** or invalid content — tell the user the server is configured
  but not responding, and go to [Troubleshooting](#troubleshooting-flow).

Describe state and actions in plain language. Do not dump raw resource output.

## Troubleshooting flow

The server is connected but not responding. Present the likely causes together —
do not walk them sequentially — and use judgement about which fits:

- **Wrong site.** The endpoint is site-specific and there is no autodetect. An
  org on EU1 talking to `mcp.datadoghq.com` authenticates against the wrong
  site. If `whoami` returned a `dd_site` that disagrees with the configured
  domain, this is the cause — go to the [Domain Flow](#domain-flow).
- **Malformed domain.** Compare against the table in `mcp-settings.md`. Flag it
  only when it looks like a typo (`mcp.us5.datadog.com`, missing the `hq`); a
  non-standard but valid domain is not automatically wrong.
- **Authentication.** The authorization may have expired or never completed. The
  user reconnects from **Settings → MCP** → **Connect** on the `datadog` server.
- **Network or access.** The network may be blocking the endpoint, or the
  account may lack API access — the `MCP Read` permission in particular.

## Domain flow

Changes the Datadog site the server talks to.

1. **Show the current setting** — from `whoami` → `dd_site` when available. Say
   it plainly ("you're currently connected to the EU1 site").
2. **Ask for the target.** Present the sites and MCP domains from
   `mcp-settings.md` and ask which to use. The user may answer with a domain, a
   site code, or an app URL — resolve it with that file's mapping rules, and ask
   rather than guess when ambiguous.
3. **Hand the change to the user.** You cannot edit the plugin's `.mcp.json`;
   see `mcp-settings.md` for why. Give them the complete URL to use, preserving
   the existing `toolsets` query string if one is set:

   ```
   https://<resolved-mcp-domain>/v1/mcp?toolsets=core,visualizations
   ```

   Tell them to put it in the `datadog` server's `url` — either in the installed
   plugin's `.mcp.json`, or by declaring their own `datadog` server in their
   user `mcp.json` — and then reconnect from **Settings → MCP**.
4. **Confirm afterwards.** When they say it is done, re-read
   `datadog://mcp/whoami` and confirm the org and `dd_site` are what they
   expected. A stale site here means the change did not take effect.

## Organization flow

Switches to a different Datadog organization. The agent cannot do this — the
user selects the organization in the browser during sign-in.

Ask whether the target organization is on the same site or a different one.

- **Same site** — they reconnect the `datadog` server from **Settings → MCP**
  and pick the target organization on the Datadog sign-in page.
- **Different site** — run the [Domain Flow](#domain-flow) first, and tell them
  to choose the target organization in the browser when they reconnect.

Either way, verify with `datadog://mcp/whoami` once they report being done.
