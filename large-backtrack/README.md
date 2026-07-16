# Large-stderr / bounded buffer (LIM-42080)

Deliberately loose upper-bound version pins on packages with wide dep-graphs so uv backtracks aggressively when resolving under a mismatched Python interpreter (e.g. `--python /python309/bin/python3.9`). In prod this pattern produces multi-GB stderr dumps.

## What the fix does

`UvPipCompiler` now caps stderr capture at 2048 lines via `RunProcessOptions.CaptureOutputMaxLines`. The unbounded `StringBuilderPipeTarget` was replaced with `CircularBufferPipeTarget` — memory usage is bounded to a few MB regardless of how much uv emits.

Before the fix: worker died with `System.OutOfMemoryException` (the analyze crashed, not just this manifest).

After the fix: the tail of uv's stderr still lands in `StandardError` (enough for error-marker matching), and the worker survives.

## What to check

- No `OutOfMemoryException` in worker logs.
- The final ~2048 lines of stderr should be visible in the exception message.
- Worker memory profile: `pip_compile_duration` counter should still fire, `pip_fallback_triggered` counter may increment.

## Note

Reproducing multi-GB stderr locally requires a genuinely pathological input. This manifest is a *plausible* trigger — actually hitting the OOM depends on the interpreter uv chooses. Success criterion: this manifest resolves (or fails gracefully) *without* crashing the worker.
