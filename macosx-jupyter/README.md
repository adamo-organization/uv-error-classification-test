# macOS/Jupyter noise files (LIM-42082 — from PR #45588)

Three tracked-in-repo accidents that used to be scanned as if they were real dependency manifests:

- `._requirements.txt` — macOS AppleDouble sidecar (binary, not UTF-8). uv would fail with `failed to decode file ... stream did not contain valid UTF-8`.
- `.ipynb_checkpoints/requirements-checkpoint.txt` — Jupyter auto-save snapshot containing pip-install flags like `--user` that uv rejects inline.
- `__MACOSX/requirements.txt` — macOS zip-archive metadata folder.

The authoritative `requirements.txt` (with `requests==2.31.0`, `flask==3.0.0`) sits next to them.

## What the fix does

`PipManifestClustersProvider` skips paths matching `(^|/)\._`, `(^|/).ipynb_checkpoints/`, or `(^|/)__MACOSX/` during manifest discovery. uv is never asked to parse them.

## What to check

- Only `macosx-jupyter/requirements.txt` should appear in the scanned dependency graph.
- `requests` and `flask` should be discovered.
- No uv errors about the three noise files.

**Note**: This validates PR #45588 (LIM-42082), which is merged separately.
