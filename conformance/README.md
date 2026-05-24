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

**v0.4 fixtures landed.** Third conformance fixture set added with the v0.4 memodef:Transcript implementation ([decisions/proposal-2026-05-20-transcript-type-and-folder-convention.md](../decisions/proposal-2026-05-20-transcript-type-and-folder-convention.md)):

**`valid_memos/` (v0.4 additions):**
- `canonical_transcript.openthing` + `canonical_transcript.body.md` — canonical v0.4 memodef:Transcript with all SHOULDs satisfied, ended absent (ongoing)
- `multi_role_transcript.openthing` + `multi_role_transcript.body.md` — three-participant transcript exercising mixed session_arc + identity entries; ended populated; related_memos[] populated
- `ongoing_transcript_no_ended.openthing` + `ongoing_transcript_no_ended.body.md` — mid-session snapshot; ended absent per SHOULD; demonstrates append-mode property

**`invalid_memos/` (v0.4 additions):**
- `transcript_without_body_ref.openthing` — body_ref MUST violation (expected: FAIL — envelope has no content reference)
- `transcript_with_body_field.openthing` — body field present on memodef:Transcript SHOULD violation (expected: Pass-with-notes per v0.3.1 latitude)
- `transcript_with_memo_fields.openthing` — memodef:Memo shape fields (from/to/action_required/in_reply_to/thread_id) on memodef:Transcript SHOULD violation (expected: Pass-with-notes per v0.3.1 latitude)

**v0.3 fixtures (still authoritative):**

**`valid_memos/` (v0.3):**
- `memo_to_file_basic.openthing` — canonical v0.3 memo-to-file with `to: "file"` sentinel
- `memo_to_file_with_threading.openthing` — successor incumbent reply to predecessor's note via `in_reply_to`
- `memo_to_file_with_body_ref.openthing` + `memo_to_file_with_body_ref.body.md` — composition with v0.2 body_ref

**`invalid_memos/` (v0.3):**
- `memo_to_file_with_action_required.openthing` — `to: "file"` + `action_required: true` semantic-mismatch SHOULD violation (expected: Pass-with-notes)

**v0.2 fixtures (still authoritative):**
- `valid_memos/with_body_ref_summary.openthing` + `with_body_ref_summary.body.md` — canonical v0.2 body_ref pair
- `valid_memos/minimal_no_body_ref.openthing` — regression guard for v0.1 inline-body shape
- `invalid_memos/body_ref_with_empty_body.openthing` — body MUST-non-empty violation (expected: FAIL)
- `invalid_memos/body_ref_subdirectory.openthing` — body_ref bare-filename SHOULD violation (expected: Pass-with-notes)

Each fixture's `metadata.fixture_purpose` documents what it tests; `metadata.fixture_validates` (valid) or `metadata.expected_validator_outcome` (invalid) document expected validator behavior. v0.4 transcript fixtures live in `valid_memos/` and `invalid_memos/` rather than a new `valid_transcripts/` subdirectory — transcripts are a class of memodef artifact and the existing conformance/ folder structure suffices. Fixtures will accumulate as the schema and template library mature.
