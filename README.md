# uv-error-classification-test

Validation repo for [apiiro/lim#45591](https://github.com/apiiro/lim/pull/45591) — uv pip-compile failure classification, bounded stderr, transient-network retry, and unsafe-best-match retry.

Each subdirectory is a self-contained `requirements.txt` scenario that isolates one of the fixes. Point an Apiiro SCA scan at this repo (or any subtree) and cross-check the log output against the per-directory README.

| Directory | Ticket | What it exercises |
|---|---|---|
| [`abi-mismatch/`](abi-mismatch/) | [LIM-42098](https://apiiro.atlassian.net/browse/LIM-42098) | `have no wheels with a matching Python ABI tag` → unactionable / Warning |
| [`url-transitive/`](url-transitive/) | [LIM-42100](https://apiiro.atlassian.net/browse/LIM-42100) | `URL dependencies must be expressed as direct requirements` → unactionable / Warning |
| [`large-backtrack/`](large-backtrack/) | [LIM-42080](https://apiiro.atlassian.net/browse/LIM-42080) | Bounded stderr buffer — worker should survive pathological resolver dumps without OOM |
| [`macosx-jupyter/`](macosx-jupyter/) | [LIM-42082](https://apiiro.atlassian.net/browse/LIM-42082) — from [#45588](https://github.com/apiiro/lim/pull/45588) | `._requirements.txt`, `.ipynb_checkpoints/`, `__MACOSX/` files should be skipped |
| [`healthy/`](healthy/) | — | Baseline — should resolve cleanly, no warnings |

## Not directly covered here

- **LIM-42084** (network retry against public PyPI hiccups) — can't reproduce a transient network failure deterministically from a requirements file. Verify with an intentional PyPI outage or by injecting a network error via a proxy.
- **LIM-42099** (uv-side stack overflow) — a bug inside uv itself; can't be triggered from a manifest reliably.
- **LIM-42098 unsafe-best-match retry** — needs a customer-side multi-index configuration (private + fallback index), not something reproducible from a public repo.

## How to validate

1. Trigger a scan against this repo in a LIM environment that has `#45591` deployed.
2. For each subdirectory, check the analyzer logs for the expected log-level classification described in that directory's README.
3. `healthy/` should complete without warnings — sanity check that the retries/classifications aren't over-firing.
