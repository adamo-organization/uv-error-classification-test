# URL-transitive dependency (LIM-42100)

`dedupe==2.0.13` transitively depends on `fastcluster` via a `git+https://github.com/dmuellner/fastcluster/archive/...zip` URL. uv's security rule blocks URL deps unless declared at the top level:

```
× Failed to resolve dependencies for `dedupe` (v2.0.13)
  ╰─▶ Package `fastcluster` was included as a URL dependency.
      URL dependencies must be expressed as direct requirements or constraints.
```

## What the fix does

`"URL dependencies must be expressed as direct requirements"` is in `UvUnactionableErrorMarkers` → logs at **Warning**, not Error. Manifest fails uv path and falls through to pip-fallback (which is more permissive) or gets dropped.

## What to check in logs

`Warning`-level `"Failed to run uv-pip-compile"` with the "URL dependencies must be expressed" marker in stderr.
