# Cloudflare

Cloudflare developer platform — 11 skills covering Workers, Durable Objects, the
Agents SDK, Wrangler, Cloudflare One and web performance, plus five remote MCP
servers for the API, docs, Workers bindings, builds and observability.

- Docs: <https://developers.cloudflare.com/agents/model-context-protocol/mcp-servers-for-cloudflare/>
- Upstream: [`cloudflare/skills`](https://github.com/cloudflare/skills) @ `30553f89` (Apache-2.0)

## The five MCP servers

Cloudflare splits its MCP surface by capability. These are independent endpoints,
not one server in five modes:

| Server | Endpoint | Auth |
|---|---|---|
| `cloudflare-api` | `mcp.cloudflare.com/mcp` | OAuth (DCR) |
| `cloudflare-docs` | `docs.mcp.cloudflare.com/mcp` | none — answers `initialize` anonymously |
| `cloudflare-bindings` | `bindings.mcp.cloudflare.com/mcp` | OAuth (DCR) |
| `cloudflare-builds` | `builds.mcp.cloudflare.com/mcp` | OAuth (DCR) |
| `cloudflare-observability` | `observability.mcp.cloudflare.com/mcp` | OAuth (DCR) |

Cloudflare runs more servers than these five (Radar, Browser Rendering, AI
Gateway, Container, …); add them to the same `mcpServers` map when wanted.

## Skills and commands

Eleven skills, vendored unmodified:

| Skill | Covers |
|---|---|
| `cloudflare` | Umbrella skill with per-product references (AI Gateway, AI Search, Analytics Engine, …) |
| `workers-best-practices` | Workers idioms and pitfalls |
| `wrangler` | The CLI — retrieval-first, because baked-in flag knowledge goes stale |
| `durable-objects` | Durable Objects |
| `agents-sdk` | Agents SDK, with references for MCP, streaming, human-in-the-loop, workflows |
| `sandbox-sdk` | Sandbox SDK |
| `cloudflare-email-service` | Email sending, routing, deliverability |
| `cloudflare-one` / `cloudflare-one-migrations` | Zero Trust and migrations |
| `web-perf` | Web performance |
| `turnstile-spin` | Turnstile |

Plus two commands: `/build-agent` and `/build-mcp`.

## Authentication

Browser-based OAuth at connect time on the four gated servers, with **no
credential to configure**. Each advertises RFC 7591 dynamic client registration
(`https://mcp.cloudflare.com/register` and the per-subdomain equivalents), so the
client mints its own public PKCE client on first authorization. `cloudflare-docs`
answers `initialize` anonymously and needs no connect step at all.

Configs are a bare `{type, url}`. No `route: "local"` — that key asserts the
provider's DCR/authorize flow accepts *only* a loopback `redirect_uri`, which has
not been established for Cloudflare, and it would be inert on the plugin path
regardless (`plugin_mcp.to_config_dict` drops it).

## Vendoring notes

This is a straight copy, in contrast to [`plugins/datadog`](../datadog/README.md),
which had to be adapted. The difference is what the skills are *about*:
Cloudflare's teach how to build on Cloudflare, so they contain no client-specific
server ids, no config-file paths, and no UI instructions. None declares
`allowed-tools`, so none of them narrows the tool list when it loads — worth
checking on any future upstream bump, since a skill's `allowed-tools` becomes a
real `tool_allowlist` here and Claude Code's PascalCase tool names would not match
ours.

`.mcp.json` is the one file replaced rather than copied: same five servers and
URLs as upstream at this commit, rewritten so it stays ours to edit. Excluded:
`.github/`, `CODEOWNERS`, `.gitignore`, `.cursor-plugin/`, and `rules/workers.mdc`
(a Cursor `.mdc` file — Corepass has no `rules` component). Full statement in
[`NOTICE`](NOTICE).

## Upgrading this entry

Re-pull upstream and re-diff `skills/` and `commands/` when its sha moves, then
bump `_provenance.sha` in the marketplace index. Check two things on every bump:
whether `.mcp.json` gained or lost a server, and whether any skill picked up an
`allowed-tools` line. Bump `version` when *our* packaging changes.
