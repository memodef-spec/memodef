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

**v0.2 fixtures landed.** First conformance fixture set added with the v0.2 `body_ref` implementation ([decisions/proposal-2026-05-01-body-ref-v0.2.md](../decisions/proposal-2026-05-01-body-ref-v0.2.md)):

**`valid_memos/`:**
- `with_body_ref_summary.openthing` + `with_body_ref_summary.body.md` — canonical v0.2 body_ref pair
- `minimal_no_body_ref.openthing` — regression guard for v0.1 inline-body shape under v0.2 readers

**`invalid_memos/`:**
- `body_ref_with_empty_body.openthing` — body MUST-non-empty violation when body_ref present (expected: FAIL)
- `body_ref_subdirectory.openthing` — body_ref bare-filename SHOULD violation (expected: Pass-with-notes)

Each fixture's `metadata.fixture_purpose` documents what it tests; `metadata.fixture_validates` (valid memos) or `metadata.fixture_violates` + `metadata.expected_validator_finding` (invalid memos) document expected validator behavior. Fixtures will accumulate as the schema and template library mature.
