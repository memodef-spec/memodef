# `memodef:Transcript` type + `transcripts/<role-id>/` folder convention (v0.4)

**Disposition:** Strategist-pending — filed by memodef-strategist; Director ratification needed before maintainer drafts SCHEMA.md text
**Origin:** Director use case surfaced 2026-05-19 (cross-runtime + cross-session role-portable VERBATIM context, distinct from memos' DISTILLED context). Designed in conversation 2026-05-19/20 after Director course-corrected an initial "separate transcriptdef-spec" sketch toward "class of memos in memodef" — keeping the artifact family unified rather than fragmenting across specs.
**Filed:** 2026-05-20 by memodef-strategist
**Schema impact:** strictly additive; minor version bump 0.3.1 → 0.4.0

## Summary

Add a new top-level type `memodef:Transcript` to memodef-spec for **verbatim conversation recordings**. Capture tools (e.g., [ccc-ninja](https://github.com/scottconfusedgorilla/ccc-ninja)) emit a `memodef:Transcript` envelope alongside a sibling `.body.md` containing the formatted markdown transcript — the body_ref pattern from v0.2 reused. Storage: `transcripts/<role-id>/` folder convention parallel to `notes/<role-id>/`. The envelope is metadata-only — no `body` field — so append-mode operation is preserved (capture tools regenerate the sibling `.body.md` from source as conversations grow; the envelope only updates `ended` on session close).

memodef-spec now spans three artifact classes serving three audiences (see Three-axis framing below).

## Motivation

Memos and memos-to-file are **distillations** — the AI's interpretation of what happened, structured for triage and retrieval. Three real use cases need the **verbatim recording**:

1. **Director snippets for books, presentations, citations.** A distilled decision artifact says "we chose X because of AI-legibility primacy"; the Director may want to quote the specific exchange where the principle was applied. The transcript is the source-of-truth.
2. **Lost-window recovery.** The Director surfaced "sometimes I lose a window and I HATE THAT" — operationally, when a Claude Code window dies mid-conversation, the context is lost. A transcript captured during the session is the recovery primitive: a fresh session can read it via `cat transcripts/<role-id>/X.body.md` and reconstruct the working state.
3. **Audit-trail pinning.** A decision artifact citing "the rationale was discussed at length" pins to a transcript filename. When the distillation is questioned, the verbatim source is queryable. The same property that v0.3's memos-to-file gave to role-incumbent successors, now layered on verbatim-not-summarized content.

The ccc-ninja VS Code extension (created by Director, pre-dates memodef) already reads Claude Code JSONL session files and produces formatted markdown with speaker labels, code blocks, and timestamps. The spec lands a typed envelope around that output so transcripts compose with the rest of the memodef family (catalog discovery, cross-references, openbraid hosted-store support, conformance fixtures).

## Proposed Change

### New top-level type (SCHEMA.md)

#### `memodef:Transcript`

A verbatim recording of a conversation between participants, captured during or after the session and stored as a catdef-family artifact. The envelope is metadata; the conversation content lives in a sibling markdown file referenced by `body_ref`.

```json
{
  "catdef": "1.4",
  "memodef": "0.4.0",
  "type": "memodef:Transcript",

  "participants": [
    {"position": "<role-id>", "session_arc": "<descriptor for AI session>"},
    {"position": "director", "identity": "<email or other identifier>"}
  ],
  "started": "<ISO 8601 timestamp>",
  "ended": "<ISO 8601 timestamp, optional>",

  "subject": "<short subject line for transcript listings>",
  "transport": "<vscode-claude-code | openbraid | claude-desktop | etc.>",
  "capture_tool": "<tool name and version, e.g. ccc-ninja@0.11.0>",
  "capture_format": "<markdown | jsonl | other>",

  "body_ref": "<sibling .body.md filename>",

  "related_memos": [
    "<filename or path of memo distilled from this transcript>"
  ],

  "metadata": {
    "redaction_status": "raw | partial | redacted",
    "retention_policy": "permanent | <other>"
  }
}
```

#### Required fields (MUST) — `memodef:Transcript`

- `catdef` (semver) — catdef substrate version
- `memodef` (semver) — memodef schema version (MUST be `"0.4.0"` or higher for this type)
- `type` (string) — MUST be exactly `"memodef:Transcript"`
- `participants` (array) — at least one participant, each with `position` (string, position id) and either `session_arc` (AI session descriptor) or `identity` (human identifier)
- `started` (ISO 8601 timestamp) — when the conversation began
- `subject` (string) — short subject line for transcript listings (scannable, ≤80 chars suggested)
- `body_ref` (string, relative path) — bare filename of the sibling `.body.md` containing the verbatim conversation content, co-located in the same directory as the envelope. The body_ref pattern from v0.2 applies; no `body` field is present on `memodef:Transcript` envelopes.

#### Recommended fields (SHOULD) — `memodef:Transcript`

- `ended` (ISO 8601 timestamp) — when the conversation closed; populate on session close if known. OPTIONAL because many conversations end without a clean close event (window died, machine restarted, session timed out).
- `transport` (string) — the substrate the conversation used (e.g., `"vscode-claude-code"`, `"openbraid"`, `"claude-desktop"`).
- `capture_tool` (string) — capture tool name and version (e.g., `"ccc-ninja@0.11.0"`).
- `capture_format` (string) — format of the body_ref content (e.g., `"markdown"`, `"jsonl"`).
- `related_memos` (array of strings) — filenames or paths of memos distilled from or referencing this transcript. Enables bidirectional audit trail: decisions cite transcripts; transcripts list the decisions they produced.
- `metadata.redaction_status` (string) — one of `"raw"` (verbatim, no redaction), `"partial"` (some content redacted), `"redaction"` (significant redaction applied). Default: `"raw"`.
- `metadata.retention_policy` (string) — `"permanent"` (default) or adopter-specific values.

#### Fields NOT present on `memodef:Transcript`

- **`body`** — `memodef:Transcript` envelopes SHOULD NOT include a `body` field. The verbatim content lives entirely in the sibling file referenced by `body_ref`. This is intentional to preserve **append-mode operation**: capture tools regenerate or append-to the sibling `.body.md` as conversations grow without ever needing to update an in-envelope abstract. Implementations encountering a `body` field on a `memodef:Transcript` SHOULD report Pass-with-notes per the v0.3.1 SHOULD-violation surfacing latitude.
- **`from`, `to`, `action_required`, `in_reply_to`, `thread_id`** — these are `memodef:Memo` shape fields. `memodef:Transcript` uses `participants[]` for multi-party authorship and has no per-recipient action or threading semantics. Per v0.3.1 SHOULD-violation latitude, implementations MAY accept these fields silently or reject; documenting which posture is recommended.

### Folder convention (README.md)

Add a new top-level section after **Notes folder — memos-to-file (RECOMMENDED, v0.3+)**:

#### Transcripts folder — verbatim recordings (RECOMMENDED, v0.4+)

A third folder convention parallel to `memos/` and `notes/<role-id>/`, for **verbatim conversation recordings**:

- `transcripts/<role-id>/` — `memodef:Transcript` envelopes + their sibling `.body.md` files (paired, both co-located)
- One folder per role: `transcripts/memodef-strategist/`, `transcripts/openbraid-engineer/`, etc.
- Multi-role conversations live in the **primary role's** folder; secondary participants appear in `participants[]` only
- Lifecycle: append-only; the sibling `.body.md` grows as the conversation progresses (capture tool regenerates or appends), the envelope's `ended` field SHOULD update once on session close, all other envelope fields are immutable after creation
- No mark-as-read — transcripts are not directed to a recipient who processes; the audience is "future readers" (Director, AI successors)

The three folder conventions now serve three audiences (see [SCHEMA.md → Three-axis framing](SCHEMA.md)):

| Axis | Type | Folder | Lifecycle | Audience |
|---|---|---|---|---|
| Inter-position coordination | `memodef:Memo` | `memos/inbox/`→`read/`→`archive/` | Maildir | AI sessions (handoffs, replies) |
| Intra-position context portability | `memodef:Memo` with `to: "file"` | `notes/<role-id>/` | Flat | Successor AI incumbents of the same role |
| Verbatim recording | `memodef:Transcript` | `transcripts/<role-id>/` | Flat, append-only | Director (snippets, recovery) + AI sessions (lost-window recovery) |

### Filename convention (SCHEMA.md §File naming convention)

`memodef:Transcript` filename pattern:

```
YYYY-MM-DD-HHMM--<primary-role-id>--<short-subject>.openthing
```

Note the deviation from `memodef:Memo` filename pattern (`YYYY-MM-DD-HHMM--<from>--<to>--<short-subject>.openthing`): transcripts have no single `from`/`to`, so the `<primary-role-id>` (the role whose folder the transcript lives in) replaces both. The full `participants[]` list lives inside the envelope.

The sibling body file follows the body_ref naming convention from v0.2: `<envelope-filename-without-.openthing>.body.md`.

### Capture-tool integration expectations (CONTRIBUTING.md or README.md)

Capture tools (e.g., ccc-ninja) SHOULD emit a `memodef:Transcript` envelope + sibling `.body.md` as an atomic pair. The envelope can be auto-populated from session metadata the tool already has access to (participants, started, transport, capture_tool, capture_format); subject and related_memos may require human or AI annotation. Tools that emit only the markdown body without an envelope produce conformant content but require manual envelope authoring by the receiver before the transcript is fully addressable.

ccc-ninja-specific integration (Director-driven separate work): a future ccc-ninja release SHOULD emit the envelope + body pair on "Save as Markdown" rather than just the markdown alone, populating envelope fields from JSONL session metadata.

## Design rationale (for the key strategist-scope calls)

### Why a new top-level type, not a sentinel on `memodef:Memo`

The v0.3 memos-to-file design (sentinel `to: "file"` + folder convention) was correct because memos-to-file are **structurally memos** — same `from`, `subject`, `sent`, `body`, optional `in_reply_to`. The Director course-corrected a too-heavy initial sketch toward POP-discipline.

Transcripts are **structurally different**:

- **Multi-participant** (no single `from` + `to`) — `participants[]` is the natural shape
- **Multi-timestamp** — `started` + `ended` rather than single `sent`
- **Append-only growing content** — no abstract that needs regeneration; `body` field would be friction
- **No action_required semantics** — transcripts have no recipient who acts
- **No threading via in_reply_to** — transcripts are continuous, not message-chained

Forcing transcript shape into `memodef:Memo` via `to: "transcript"` sentinel would distort five fields' semantics. A new top-level type within memodef-spec is the right shape — precedent exists (`memodef:Memo` + `memodef:Library` are already two types).

### Why no `body` field on `memodef:Transcript`

Append-mode operation is the load-bearing property. The Director explicitly identified abstract-regeneration as friction: every time the conversation grows, an inline body abstract would need re-summarization. With body omitted, the envelope is set ONCE on transcript creation and the sibling `.body.md` grows by regeneration or append. The envelope's only mutable field is `ended`, set once at close.

This is the inverse of v0.2's body_ref design for `memodef:Memo`: there, body was MUST-non-empty as an AI-triage abstract because the envelope is the scannable artifact. Here, the audience is primarily Director (snippets, recovery, citation) and successor AI sessions doing context-rebuild, not triage. The triage purpose body served for memos doesn't apply to transcripts.

### Why `transcripts/<role-id>/` parallel to `notes/<role-id>/`

Three folder conventions parallel each other. Per-role flat. Multi-role transcripts pick a primary-role folder; secondary participants appear in `participants[]`. This matches the v0.3 design's posture on cross-role concerns (deferred — not memodef's primary concern).

Alternative considered: `transcripts/<conversation-id>/` keyed by an opaque id, allowing transcripts to live "between" roles. Rejected — the existing per-role flat pattern composes more naturally and avoids the "who allocates conversation ids" infrastructure question.

### Why markdown as canonical body content (via body_ref)

ccc-ninja already produces formatted markdown from JSONL — speaker labels, code blocks, timestamps. Markdown is the durable form (human-readable, paste-friendly for snippets, survives format changes, exposes the conversation's literal text). JSONL is the unstable substrate (Anthropic-controlled, format may evolve). The spec recommends markdown as the canonical body content via the `capture_format` field; adopters MAY use other formats (jsonl, plain text) for tools that emit those.

### Why ccc-ninja integration is "SHOULD emit envelope," not "spec mandates capture tool"

memodef-spec does not specify capture tools. ccc-ninja is named as a reference implementation in the same posture as Claude Code Channels were named in the v0.1 notification section: documenting the canonical instantiation without preferring a vendor. Other capture tools (openbraid-side capture, future Claude Code native, third-party) are equally valid; the spec defines the artifact SHAPE, not the tool that produces it.

## Backward Compatibility

Strictly additive. No existing `memodef:Memo` or `memodef:Library` artifact becomes invalid. v0.1 / v0.2 / v0.3 / v0.3.1 readers may encounter `memodef:Transcript` artifacts and SHOULD report Pass-with-notes ("unknown type") per catdef's reader-lenient discipline rather than hard fail. `memodef:Transcript` artifacts MUST declare `"memodef": "0.4.0"` or higher per the version-stamp-consistent-with-features-used rule.

`memos/` and `notes/<role-id>/` folder conventions are unchanged. The new `transcripts/<role-id>/` is parallel and additive.

The body_ref pattern from v0.2 is reused; no new sibling-file convention is introduced. Capture tools producing `memodef:Transcript` envelope + sibling `.body.md` pairs use the same atomic-pair semantics as v0.2 memos with body_ref.

## Conformance Tests

To add to `conformance/`:

- **`valid_memos/canonical_transcript.openthing`** + **`canonical_transcript.body.md`** — canonical v0.4 envelope with sibling markdown body; demonstrates the atomic pair pattern.
- **`valid_memos/multi_role_transcript.openthing`** — envelope with multiple participants (e.g., memodef-strategist + openbraid-engineer + director); demonstrates participants[] with mixed AI session_arc and human identity.
- **`valid_memos/ongoing_transcript_no_ended.openthing`** — envelope with no `ended` field (mid-conversation snapshot); demonstrates that ended is OPTIONAL.
- **`invalid_memos/transcript_without_body_ref.openthing`** — envelope missing `body_ref` field; expected FAIL (body_ref is MUST for `memodef:Transcript` since no body field exists).
- **`invalid_memos/transcript_with_body_field.openthing`** — envelope with a `body` field present; expected Pass-with-notes (SHOULD NOT include body per the no-abstract design rationale; v0.3.1 latitude allows implementations to choose strict-or-lenient).
- **`invalid_memos/transcript_with_memo_fields.openthing`** — envelope with `from`/`to`/`action_required` fields (memodef:Memo shape fields on a memodef:Transcript); expected Pass-with-notes per the same v0.3.1 latitude.

Fixtures live in `conformance/valid_memos/` and `conformance/invalid_memos/` rather than a new `valid_transcripts/` subdirectory, since transcripts are a class of memodef artifact and the existing conformance/ folder structure suffices for v0.4.

## Alternatives Considered

- **Separate sibling spec `transcriptdef-spec`.** Director course-corrected: "I truly think this is a class of memos, and should be in memodef." Keeping the artifact family unified under one spec is the right shape; transcriptdef-spec would fragment the audience and double the governance/conformance surface.
- **Sentinel `to: "transcript"` on `memodef:Memo`.** Rejected per Design rationale — structural shape genuinely different.
- **`body` field as short subject-shaped abstract.** Rejected per Director's no-abstract rule and append-mode requirement.
- **JSONL as canonical body content.** Rejected — markdown is durable; JSONL is unstable substrate.
- **`transcripts/<conversation-id>/` per-conversation folder.** Rejected — per-role parallel to notes/ composes more naturally.
- **`abstract` field as OPTIONAL short summary.** Rejected — adopters who need a one-line description can put it in `subject`. Adding a separate abstract field re-introduces the regeneration-friction the Director explicitly identified.
- **Spec ships capture tooling.** Rejected — memodef defines artifact SHAPE, not capture tool. ccc-ninja and future capture tools are reference implementations, not spec deliverables.
- **`participants[]` with detailed schema for each entry.** Considered — could include start_time, end_time, role_at_time, etc. per participant. Out of scope for v0.4; defer until empirical motivation. Current schema is loosely typed `{position, session_arc OR identity}` which adopters can extend via metadata.

## Open Questions

For Director ratification:

1. **`body` field on `memodef:Transcript`: SHOULD NOT vs MUST NOT.** Current lean: SHOULD NOT (Pass-with-notes if present) per v0.3.1 SHOULD-violation latitude. MUST NOT would be stricter (FAIL if present) but matches the design intent more closely. Adopters who include a body summary aren't actively harmful; they're just over-engineering by their own choice.
2. **`body_ref` MUST vs SHOULD for `memodef:Transcript`.** Current proposal: MUST. Without body and without body_ref, the envelope has no content reference. Could relax to SHOULD if adopters want envelope-only "stub" transcripts (rejected as edge-case; MUST is cleaner).
3. **`ended` field semantics on close.** Current: SHOULD-fill at session close. Some transports (openbraid) can auto-detect close; others (vscode + ccc-ninja) require explicit user action. Confirm SHOULD level (not MUST) given variability.
4. **Filename pattern.** Current: `YYYY-MM-DD-HHMM--<primary-role-id>--<short-subject>.openthing`. Deviation from `memodef:Memo`'s `<from>--<to>--<subject>` pattern. Acceptable, or prefer `<role-id>--transcript--<subject>` for visual parallelism with `<from>--<to>--<subject>`?
5. **`metadata.redaction_status` enum values.** Current: `raw | partial | redacted`. Sufficient, or anticipate more (e.g., `pre-redaction-review`, `auto-redacted`, `human-reviewed`)? Lean: keep three; let adopters extend via `x.<domain>.*` extensions.
6. **`related_memos[]` path resolution.** Current: bare filenames OR full paths (loosely typed). Should it be normatively specified (bare filename only? paths permitted?)? Lean: keep loose for v0.4; the field is informational, not behavior-driving.
7. **openbraid API scoping.** Same posture as v0.3 Q5: openbraid's call, not normative for memodef. Whether `send_memo` extends to transcripts via `kind="transcript"` discriminator or a parallel `send_transcript` MCP tool is openbraid's architectural decision. Flag for tracking only.
8. **ccc-ninja integration.** The Director has indicated he'll prompt the ccc-ninja session (which predates memos) separately to do the envelope-emission work. Spec-side language SHOULD encourage capture tools to emit envelopes; ccc-ninja-specific work is out-of-scope for this proposal.

## Cross-references

- v0.3 memos-to-file decision (sentinel-and-folder precedent, POP-discipline operational test): [decisions/proposal-2026-05-10-memos-to-file-and-notes-folder.md](../decisions/proposal-2026-05-10-memos-to-file-and-notes-folder.md)
- v0.2 body_ref decision (sibling .body.md pattern, atomic-move helper): [decisions/proposal-2026-05-01-body-ref-v0.2.md](../decisions/proposal-2026-05-01-body-ref-v0.2.md)
- v0.3.1 implementer-experience clarifications (SHOULD-violation surfacing latitude, hosted-store metadata passthrough): [decisions/v0.3.1-implementer-experience-clarifications.md](../decisions/v0.3.1-implementer-experience-clarifications.md)
- Charter (AI-legibility primacy, POP-discipline, no-vendor-advocacy values): [org/memodef-spec-organization.opencatalog](../org/memodef-spec-organization.opencatalog)
- Architecture-validation decision (folder-as-state, README-vs-SCHEMA placement precedent): [decisions/proposal-2026-04-29-catdef-strategist-architecture-validation.md](../decisions/proposal-2026-04-29-catdef-strategist-architecture-validation.md)
- ccc-ninja capture tool (reference implementation in flight; Director-driven envelope-emission work separate from this proposal): https://github.com/scottconfusedgorilla/ccc-ninja
- openbraid hosted-store (potential transport for openbraid-mediated transcripts; API scoping is openbraid's call per Q7): https://openbraid.app
