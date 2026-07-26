# corepass-plugins

Two independent distribution tracks live in this repo. They share nothing but
the repo — a plugin may appear in either, both, or neither.

| Track | Entry point | Content | Maintenance |
|---|---|---|---|
| **Catalog** | [`catalog_seed.json`](catalog_seed.json) | git coords only, nothing vendored | generated |
| **Marketplace** | [`.corepass-plugin/marketplace.json`](.corepass-plugin/marketplace.json) | plugin files vendored under `plugins/` | hand-maintained |

## Track 1 — hosted catalog (generated)

A normalized directory of open-source Claude Code plugins, backing the app's
Plugins storefront.

- **Consumed remotely**: the app fetches the raw file at
  `https://raw.githubusercontent.com/youlandinc/plugins/main/catalog_seed.json`
  (via `COREPASS_CATALOG_URL`), so the plugin list updates without shipping a
  new app build.
- Each entry carries install git coords (`source.url` / `source.ref` /
  `source.path`, pointing at each plugin's own upstream repo) plus enumerated
  components (`skills` / `mcpServers` / `commands` / `agents` / …).
  Install/uninstall clone those coords through the existing catalog path —
  **nothing is vendored for this track**.
- **Generated**, do not hand-edit. Produced by `scripts/normalize_plugins.py`
  in the `simple-coding-harness` repo.

### Regenerate

```bash
python scripts/normalize_plugins.py --emit catalog-seed \
  --out /path/to/plugins/catalog_seed.json \
  --all --require-license \
  --merge-into src/registry/data/catalog_seed.json
# review, then commit + push
```

Only plugins that ship a `LICENSE` file are included (`--require-license`);
each entry records its upstream url + pinned rev under `_provenance`.
Third-party plugins remain under their own licenses.

## Track 2 — marketplace (hand-maintained, vendored)

Layout mirrors [`youlandinc/plugin-template`](https://github.com/youlandinc/plugin-template):
a storefront index at `.corepass-plugin/marketplace.json` plus one vendored
directory per plugin under `plugins/<name>/`.

```
.corepass-plugin/marketplace.json     # storefront index
plugins/
  slack/
    .corepass-plugin/plugin.json      # canonical manifest (ours)
    .claude-plugin/plugin.json        # upstream manifests, kept as-is
    .cursor-plugin/plugin.json
    .mcp.json
    skills/<name>/SKILL.md
    commands/*.md
    LICENSE
  figma/
    .corepass-plugin/plugin.json
    .mcp.json
    skills/<name>/SKILL.md
    Figma Icon (Full-color).svg
    NOTICE.md                       # no upstream LICENSE — see Licensing below
  sfranalytics/
    .corepass-plugin/plugin.json      # authored here, not vendored
    .mcp.json                         # one hosted MCP server, no other components
  superpowers/
    .corepass-plugin/plugin.json
    .claude-plugin/plugin.json
    .cursor-plugin/plugin.json
    skills/<name>/SKILL.md
    hooks/{hooks.json,run-hook.cmd,session-start}
    assets/superpowers-small.svg
    LICENSE
```

Install them:

```bash
harness marketplace add https://github.com/youlandinc/plugins --name corepass-plugins
harness plugin install slack@corepass-plugins
harness plugin install figma@corepass-plugins
harness plugin install sfranalytics@corepass-plugins
harness plugin install superpowers@corepass-plugins
```

### Why vendored

A marketplace entry's `source` must resolve to a directory **inside this repo** —
`src/registry/marketplace.py::_entry_source_path` accepts only `"./plugins/x"` or
`{"path": "plugins/x"}`, never an external git coord. Pointing an entry at
another repo is therefore not possible; the files have to live here.

### Adding a plugin

1. Fork the upstream repo into `youlandinc/` so the mirror is ours and upstream
   changes can be merged rather than re-diffed by hand.
2. Copy only the plugin **runtime** content into `plugins/<name>/` — manifest,
   `skills/`, `commands/`, `agents/`, `hooks/`, MCP config, `LICENSE`, `README`.
   Leave CI, changesets, tests and build tooling behind.
3. Leave the upstream manifests (`.claude-plugin/`, `.cursor-plugin/`) **as-is**
   so the copy stays diffable against upstream, and add our own
   `.corepass-plugin/plugin.json`. The harness probes `.corepass-plugin/`,
   `.claude-plugin/`, then `.cursor-plugin/` and uses the first hit, so ours
   wins. Upstream manifests routinely omit `displayName` — without it the
   storefront falls back to the bare slug.
4. **Declare `hooks` explicitly.** `plugin_discovery::_conventional` adopts a
   conventional subdirectory for `skills` / `commands` / `agents` / `tools` when
   the manifest leaves the list undeclared, and `plugin_mcp` does the same for a
   plugin-root `mcp.json` / `.mcp.json` — but **nothing does it for hooks**.
   Claude Code loads `hooks/hooks.json` by convention, so an upstream plugin
   that relies on that (superpowers does) silently loses its hooks here unless
   `.corepass-plugin/plugin.json` says `"hooks": "./hooks/hooks.json"`.
5. Add the entry to `.corepass-plugin/marketplace.json`, recording the fork and
   the exact vendored `sha` under `_provenance`.

Not every entry is vendored from somewhere. A vendor that ships only a hosted MCP
endpoint and no plugin repository (`sfranalytics`) gets its manifest and
`.mcp.json` authored here instead, and carries **no** `_provenance` — there is no
upstream commit to pin, and nothing to re-sync. Steps 1 and 2 simply do not
apply; `version` in such a manifest tracks our packaging, not the vendor's API.

This is also the only way to list a hosted-only connector at all: the catalog
track requires a git `source.url`, which such a vendor does not have.

### Licensing

`slack` and `superpowers` are MIT and each keeps its `LICENSE` file alongside its
files — the normal case.

`figma` is the exception and is flagged as such: `figma/mcp-server-guide` ships
**no** `LICENSE` (GitHub reports `license: null`) and its README places the
skills under the [Figma Developer Terms](https://www.figma.com/legal/developer-terms/)
as a Beta feature. It is vendored anyway, because the marketplace track has no
way to list a plugin without a copy in this repo, and the status is recorded in
[`plugins/figma/NOTICE.md`](plugins/figma/NOTICE.md) plus `license: null` in its
`_provenance`. Nothing there is relicensed.

If that copy ever has to go, the catalog track is the drop-in replacement: it
stores git coordinates only and the user clones from Figma's own repo, so the
plugin stays installable. Prefer catalog-only for any future plugin without a
redistributable license — vendor it only when the marketplace listing is
genuinely needed, and add a `NOTICE.md` when you do.
