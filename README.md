# youland-plugins

A self-contained mirror of open-source Claude Code plugins, normalized so every
plugin is a **local `./plugins/<name>` subdirectory**. This lets the Corepass
harness install/uninstall them through its existing local-source path with no
runtime changes.

- **Generated**, do not hand-edit. Produced by
  `scripts/normalize_plugins.py` in the `simple-coding-harness` repo.
- **Source**: upstream [`anthropics/claude-plugins-official`](https://github.com/anthropics/claude-plugins-official).
- **Scope**: only plugins that ship a `LICENSE`/`COPYING` file are included
  (`--require-license`). Each entry records its upstream `url` + pinned `rev`
  under `_provenance` in `.claude-plugin/marketplace.json`.

## Regenerate

```bash
python scripts/normalize_plugins.py --out /path/to/plugins --all --require-license --prune
```

Third-party plugins remain under their own licenses; see each
`plugins/<name>/LICENSE`.
