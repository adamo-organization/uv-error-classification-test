# ABI-mismatch (LIM-42098)

`pyodbc==4.0.28` publishes wheels only for older CPython ABI tags. When uv is invoked with `--python /python313/bin/python3.13`, it produces:

```
× No solution found when resolving dependencies:
  ╰─▶ ... have no wheels with a matching Python ABI tag (e.g., `cp313`), we can conclude that ...
hint: You require CPython 3.13 (`cp313`), but we only found wheels for `pyodbc` (v4.0.28) with the following Python ABI tag: `cp39`
```

## What the fix does

`"have no wheels with a matching Python ABI tag"` is now in `UvUnactionableErrorMarkers` → this failure logs at **Warning** level instead of Error, so it stops polluting the error dashboard.

## What to check in logs

Look for a `Warning`-level `"Failed to run uv-pip-compile"` entry with the ABI-tag hint in stderr. Before the fix it would have been `Error`-level.
