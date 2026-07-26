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
    .claude-plugin/plugin.json        # upstream manifest, kept as-is
    .mcp.json
    skills/<name>/SKILL.md
    commands/*.md
    LICENSE
```

Install it:

```bash
harness marketplace add https://github.com/youlandinc/plugins --name corepass-plugins
harness plugin install slack@corepass-plugins
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
3. Keep the upstream plugin manifest **as-is** (`.claude-plugin/plugin.json`
   loads fine — the harness probes `.corepass-plugin/`, `.claude-plugin/`, then
   `.cursor-plugin/`). Not duplicating it into `.corepass-plugin/` is what keeps
   the vendored copy diffable against upstream.
4. Add the entry to `.corepass-plugin/marketplace.json`, recording the fork and
   the exact vendored `sha` under `_provenance`.

Vendored plugins remain under their own upstream licenses; each keeps its
`LICENSE` file alongside its files.
