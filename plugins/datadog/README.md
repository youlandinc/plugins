# Datadog

Datadog integration — query metrics, logs, traces, monitors, dashboards, and
incidents through Bits AI's remote MCP server.

- Docs: <https://docs.datadoghq.com/bits_ai/mcp_server/>
- Endpoint: `https://mcp.datadoghq.com/v1/mcp?toolsets=core,visualizations` (Streamable HTTP)
- Upstream: [`datadog-labs/cursor-plugin`](https://github.com/datadog-labs/cursor-plugin) @ `7136415` (Apache-2.0)

## What this plugin is

One MCP server plus three skills adapted from upstream:

```json
{
  "mcpServers": {
    "datadog": {
      "type": "http",
      "url": "https://mcp.datadoghq.com/v1/mcp?toolsets=core,visualizations"
    }
  }
}
```

| Skill | Purpose |
|---|---|
| `ddsetup` | Routes Datadog questions to the MCP tools; gets the server connected when it is not, instead of falling back to the web UI |
| `ddconfig` | Shows the current org/site via `whoami`, changes site, switches organization, troubleshoots a connected-but-dead server |
| `ddtoolsets` | Lists toolsets from `datadog://mcp/toolsets` and computes a new `toolsets=` value |

## Authentication

Browser-based OAuth at connect time, with **no credential to configure**.
Datadog advertises RFC 7591 dynamic client registration
(`https://app.datadoghq.com/api/v2/oauth2/register`), so the client registers its
own public PKCE client during the first authorization.

Upstream's `.dd_cursor_mcp.json` also carries `DD_API_KEY` /
`DD_APPLICATION_KEY` headers, which is easy to misread as the primary path. It
is not: upstream's own reference states the server "uses OAuth by default, and
API keys are for advanced usage". The adapted skills never ask for a key.

## Site / region

The endpoint is **site-specific** and there is no autodetect. `mcp.datadoghq.com`
is US1; other sites need `mcp.us3.datadoghq.com`, `mcp.us5.datadoghq.com`,
`mcp.datadoghq.eu`, `mcp.ap1.datadoghq.com`, or `mcp.ap2.datadoghq.com`. The
`ddconfig` skill walks a user through switching. `toolsets=core,visualizations`
mirrors upstream's default; drop the query string to follow the server's.

## What was adapted, and why

Upstream is a Cursor plugin in the literal sense. Three of its mechanics do not
survive the move, and the skills were rewritten around that rather than dropped
(full statement of changes in [`NOTICE`](NOTICE)):

- **URL templating.** Upstream builds the URL from `${DD_MCP_DOMAIN:-not-setup}`.
  `transports/headers._expand_env` handles `${VAR}` and `${VAR:-default}`, but it
  runs on **headers only** — `config["url"]` reaches the transport verbatim, so
  the placeholder would ship as a literal hostname. Our `.mcp.json` therefore
  carries a resolved URL.
- **Unset-variable headers.** `${DD_API_KEY}` has no `:-` default, and
  `_expand_env` raises on an unset variable with no default. Vendored as-is, the
  ordinary OAuth case (no API keys exported) would fail at connect with an
  exception. The header pair is omitted.
- **Self-editing config.** All three upstream skills read and rewrite
  `.dd_cursor_mcp.json` to set the site and toolsets. An agent here cannot do
  that: the installed plugin lives in the Corepass project store, outside the
  workspace the file tools can reach, and `plugins/` is a reserved path in a
  managed workspace. The skills now compute the correct URL and hand it to the
  user, who applies it and reconnects.

Also adapted: the server is addressed as `datadog__datadog` (tools
`mcp__datadog__datadog__*`), not Cursor's `plugin-datadog-datadog`, and Cursor
command-palette instructions are replaced with **Settings → MCP**.

Not vendored: `.github/` (CI, release workflow, issue and PR templates,
dependabot, CODEOWNERS) and `CONTRIBUTING.md` — repository tooling, excluded per
this repo's "leave CI and build tooling behind" rule. `.cursor-plugin/plugin.json`
and `.dd_cursor_mcp.json` are also left out: that manifest points at a
registration file we do not ship, so keeping it would be a dangling reference
rather than a useful upstream diff.

## Upgrading this entry

Two independent axes. Re-read upstream when its `sha` moves and re-apply the
adaptations above; bump `version` when *our* packaging changes (a new URL, an
added server). If upstream ever stops being Cursor-shaped — real `${}` support in
URLs, or client-neutral skills — replace the adaptation with a straight vendored
copy.
