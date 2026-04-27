# conformance/

Conformance fixtures and runtime-evidence artifacts for memodef.

## Subdirectories

### `valid_memos/`

Exemplar `memodef:Memo` artifacts demonstrating proper schema usage. Each fixture exercises a specific aspect of the schema (full-feature memo, minimal-required-only memo, threading via `in_reply_to`, broadcast addressing, legacy `x.memo.*` form for backward-compat verification).

### `invalid_memos/`

Counterexamples demonstrating common errors validators must catch (missing required fields, malformed `sent` timestamps, reserved-namespace misuse, etc.).

### `runtime_evidence/`

Per-runtime test outputs documenting how a given runtime behaves with memodef artifacts. v0.2+ work; deferred until memodef-aware tooling lands.

## Status

**Empty.** v0.1 bootstrap. Fixtures will accumulate as the schema and the template library mature.
