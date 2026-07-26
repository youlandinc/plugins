# NOTICE — figma

**This directory carries no open-source license.** Unlike every other vendored
plugin here, it is not MIT or similar, and there is no `LICENSE` file to keep
alongside it — because the upstream repository does not ship one.

## Status

- **Upstream**: <https://github.com/figma/mcp-server-guide>
- **Vendored from**: <https://github.com/youlandinc/mcp-server-guide> (our fork)
  @ `07316dd2920d61303ca0e52812b31f5f341e7b15`
- **Upstream license file**: none. GitHub reports the repository's license as
  `null`.
- **Applicable terms**: the upstream `README.md` states that use of the Figma MCP
  server and its related resources, *including these skills*, is subject to the
  [Figma Developer Terms](https://www.figma.com/legal/developer-terms/), and that
  the skills are a **Beta** feature.

All content in this directory remains the copyright of Figma, Inc. Nothing here
is relicensed, and the presence of this copy grants no rights that the Figma
Developer Terms do not already grant.

## Why the copy exists

A marketplace entry's `source` must resolve to a directory inside this
repository — `marketplace.py::_entry_source_path` accepts only `"./plugins/x"`
or `{"path": "plugins/x"}`, and the marketplace clone is not recursive, so a git
submodule would arrive empty. There is no way to list a plugin on the
marketplace track without a copy living here.

The **catalog** track (`catalog_seed.json`) has no such constraint: it stores git
coordinates only and the user clones from Figma's own repository. If this copy
ever needs to go away, the catalog entry is the drop-in replacement — the plugin
stays installable, just through Figma's repo instead of ours.

## On request

Remove this directory and its entry from `.corepass-plugin/marketplace.json` if
Figma asks, or if the Developer Terms are read to disallow redistribution. Doing
so costs nothing beyond the marketplace listing.
