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

**v0.3 fixtures landed.** Second conformance fixture set added with the v0.3 memos-to-file implementation ([decisions/proposal-2026-05-10-memos-to-file-and-notes-folder.md](../decisions/proposal-2026-05-10-memos-to-file-and-notes-folder.md)):

**`valid_memos/` (v0.3 additions):**
- `memo_to_file_basic.openthing` — canonical v0.3 memo-to-file with `to: "file"` sentinel
- `memo_to_file_with_threading.openthing` — successor incumbent reply to predecessor's note via `in_reply_to`
- `memo_to_file_with_body_ref.openthing` + `memo_to_file_with_body_ref.body.md` — composition with v0.2 body_ref

**`invalid_memos/` (v0.3 additions):**
- `memo_to_file_with_action_required.openthing` — `to: "file"` + `action_required: true` semantic-mismatch SHOULD violation (expected: Pass-with-notes)

**v0.2 fixtures (still authoritative):**
- `valid_memos/with_body_ref_summary.openthing` + `with_body_ref_summary.body.md` — canonical v0.2 body_ref pair
- `valid_memos/minimal_no_body_ref.openthing` — regression guard for v0.1 inline-body shape
- `invalid_memos/body_ref_with_empty_body.openthing` — body MUST-non-empty violation (expected: FAIL)
- `invalid_memos/body_ref_subdirectory.openthing` — body_ref bare-filename SHOULD violation (expected: Pass-with-notes)

Each fixture's `metadata.fixture_purpose` documents what it tests; `metadata.fixture_validates` (valid memos) or `metadata.fixture_violates` + `metadata.expected_validator_finding` (invalid memos) document expected validator behavior. v0.3 fixtures additionally document `metadata.intended_folder_placement` since fixtures live in `conformance/` not in their natural folder positions. Fixtures will accumulate as the schema and template library mature.
