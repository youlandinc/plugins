# corepass-plugins

Hosted plugin **catalog** for the Corepass harness. The single source of truth
is [`catalog_seed.json`](catalog_seed.json) — a normalized directory of
open-source Claude Code plugins.

- **Consumed remotely**: the app fetches the raw file at
  `https://raw.githubusercontent.com/youlandinc/plugins/main/catalog_seed.json`
  (via `COREPASS_CATALOG_URL`), so the plugin list updates without shipping a
  new app build.
- Each entry carries install git coords (`gitUrl` / `gitRef` / `gitPath`,
  pointing at each plugin's own upstream repo) plus enumerated components
  (`skills` / `mcpServers` / `commands` / `subagents`). Install/uninstall clone
  those coords through the existing catalog path — nothing is vendored here.
- **Generated**, do not hand-edit. Produced by `scripts/normalize_plugins.py`
  in the `simple-coding-harness` repo.

## Regenerate

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
