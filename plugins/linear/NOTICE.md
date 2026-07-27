# NOTICE — linear

**This directory carries no open-source license.** Like `figma`, it is not MIT
or similar, and there is no `LICENSE` file to keep alongside it — because the
upstream repository does not ship one.

## Status

- **Upstream**: <https://github.com/linear/cursor-plugin>
- **Vendored from**: <https://github.com/youlandinc/cursor-plugin> (our fork)
  @ `c2c4cb2ab23206c9219b0dd31c9571e4c922faeb`
- **Upstream license file**: none. GitHub reports the repository's license as
  `null`.

All content in this directory remains the copyright of Linear. Nothing here is
relicensed, and the presence of this copy grants no rights beyond what Linear's
own terms of service already grant.

## Why the copy exists

A marketplace entry's `source` must resolve to a directory inside this
repository — `marketplace.py::_entry_source_path` accepts only `"./plugins/x"`
or `{"path": "plugins/x"}`, and the marketplace clone is not recursive, so a git
submodule would arrive empty. There is no way to list a plugin on the
marketplace track without a copy living here.

## On request

Remove this directory and its entry from `.corepass-plugin/marketplace.json` if
Linear asks.
