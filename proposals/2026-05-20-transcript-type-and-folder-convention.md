# `memodef:Transcript` type + `transcripts/<role-id>/` folder convention (v0.4)

**Disposition:** Strategist-pending — filed by memodef-strategist; Director ratification needed before maintainer drafts SCHEMA.md text
**Origin:** Director use case surfaced 2026-05-19 (cross-runtime + cross-session role-portable VERBATIM context, distinct from memos' DISTILLED context). Designed in conversation 2026-05-19/20 after Director course-corrected an initial "separate transcriptdef-spec" sketch toward "class of memos in memodef" — keeping the artifact family unified rather than fragmenting across specs.
**Filed:** 2026-05-20 by memodef-strategist
**Revised:** 2026-05-20 by memodef-strategist — first-implementation observations from ccc-ninja@0.12.0 capture folded back in (folder-root anchoring made explicit; `ended` semantics tightened; subject-quality guidance added; OQ3 and OQ5 resolved; new OQ on folder-root anchoring added). See "First-implementation observations" section.
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

- `ended` (ISO 8601 timestamp) — when the conversation closed. **SHOULD be ABSENT while the conversation is ongoing or its close cannot be detected.** SHOULD be populated exactly once on session close if known. Mid-session snapshots SHOULD omit `ended`; setting `ended` to a snapshot-time falsely claims conversation termination and conflicts with the append-mode-is-load-bearing rationale (`ended` is the envelope's only mutable field; mutating it on every snapshot defeats the "envelope set once, body grows" property). Many conversations end without a clean close event (window died, machine restarted, session timed out) — in those cases `ended` remains absent permanently.
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
- The `transcripts/` folder lives at the **working-repo root** — the same directory level as `CLAUDE.md`, `SCHEMA.md`, `README.md`, `memos/`, `decisions/`, `org/`. In a repo where the memodef working directory is nested below the GitHub-repo root (e.g., `memodef-spec/memodef/` inside the `memodef-spec` GitHub repo), `transcripts/` lives at `memodef-spec/memodef/transcripts/`, NOT at `memodef-spec/transcripts/`. This matches the placement of `memos/` and `notes/<role-id>/` — per-working-repo, not per-GitHub-repo. Capture tools writing into a working repo MUST resolve the working-repo root (the directory containing `CLAUDE.md` or `SCHEMA.md`) before writing transcripts.
- One folder per role: `transcripts/memodef-strategist/`, `transcripts/openbraid-engineer/`, etc.
- Multi-role conversations live in the **primary role's** folder; secondary participants appear in `participants[]` only
- Lifecycle: append-only; the sibling `.body.md` grows as the conversation progresses (capture tool regenerates or appends), the envelope's `ended` field is set exactly once on session close and is otherwise absent (see SHOULD rules below), all other envelope fields are immutable after creation
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

Capture tools SHOULD derive `subject` from substantive conversational content — typically the first user turn after harness/system bootstrap — and not from harness metadata, transport-injected tags (e.g., `<ide_opened_file>`, `<system-reminder>`, system messages emitted by the runtime before the user's first turn), or capture-tool internals. The `subject` is the primary scannable identifier in transcript listings and the seed for the filename slug; noise in the subject propagates into the filename and degrades discoverability for the audiences this artifact class serves (Director snippet-search, lost-window recovery, audit-trail pinning). This is capture-tool quality guidance, not a SCHEMA MUST: tools that emit noisy subjects produce conformant but degraded transcripts.

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
3. ~~**`ended` field semantics on close.**~~ **RESOLVED 2026-05-20 (revision)** by ccc-ninja@0.12.0 first-implementation evidence — see "First-implementation observations" section. Resolution: `ended` SHOULD be ABSENT while the conversation is ongoing or its close cannot be detected; SHOULD be populated exactly once on session close. Mid-session snapshots SHOULD omit `ended`. SHOULD level (not MUST) confirmed given close-detection variability across transports. Reflected in SCHEMA SHOULD fields above.
4. **Filename pattern.** Current: `YYYY-MM-DD-HHMM--<primary-role-id>--<short-subject>.openthing`. Deviation from `memodef:Memo`'s `<from>--<to>--<subject>` pattern. Acceptable, or prefer `<role-id>--transcript--<subject>` for visual parallelism with `<from>--<to>--<subject>`?
5. ~~**`metadata.redaction_status` enum values.**~~ **RESOLVED 2026-05-20 (revision)** by ccc-ninja@0.12.0 first-implementation evidence — ccc-ninja emitted `"raw"` cleanly; the three-value enum (`raw | partial | redacted`) is sufficient. Adopters needing more granularity extend via `x.<domain>.*`.
6. **`related_memos[]` path resolution.** Current: bare filenames OR full paths (loosely typed). Should it be normatively specified (bare filename only? paths permitted?)? Lean: keep loose for v0.4; the field is informational, not behavior-driving.
7. **openbraid API scoping.** Same posture as v0.3 Q5: openbraid's call, not normative for memodef. Whether `send_memo` extends to transcripts via `kind="transcript"` discriminator or a parallel `send_transcript` MCP tool is openbraid's architectural decision. Flag for tracking only.
8. **ccc-ninja integration.** The Director has indicated he'll prompt the ccc-ninja session (which predates memos) separately to do the envelope-emission work. Spec-side language SHOULD encourage capture tools to emit envelopes; ccc-ninja-specific work is out-of-scope for this proposal.

9. **NEW — Folder-root anchoring for `transcripts/`.** Surfaced by ccc-ninja@0.12.0's first capture, which placed transcripts at the GitHub-repo root (`memodef-spec/transcripts/`) rather than the working-repo root (`memodef-spec/memodef/transcripts/`). The original proposal text said "parallel to `notes/<role-id>/`" without explicitly specifying which root — a genuine ambiguity when the memodef working directory is nested inside a wrapping GitHub repo. Current revision: working-repo root (the directory containing `CLAUDE.md` / `SCHEMA.md`), matching the placement of `memos/` and `notes/`. Reflected in the Folder convention section above. Confirm at ratification.

## First-implementation observations

A first implementation of this proposal shipped as a side effect of writing it: the ccc-ninja VS Code extension at version 0.12.0 began emitting `memodef:Transcript` envelope + sibling `.body.md` pairs ahead of Director ratification, capturing the memodef-strategist session conducting this proposal's revision (see `transcripts/memodef-strategist/2026-05-20-1932--memodef-strategist--ide-opened-file-the-user-opened-the-file-untitled.{openthing,body.md}`). The capture mirrors the openbraid-engineer 2026-05-10 implementation-experience pattern that drove v0.3.1: first-reader experience folded back into the proposal pre-ratification, strengthening rather than fragmenting the artifact.

What the implementation got right (v0.4 conformance confirmation):

- `type: "memodef:Transcript"`, `memodef: "0.4.0"` stamp
- `participants[]` single-entry intra-position shape, with `position` + free-text `session_arc`
- `started` / `ended` ISO timestamps populated
- `body_ref` to sibling `.body.md`; envelope has no `body` field (append-mode preserved)
- `capture_tool: "ccc-ninja@0.12.0"`, `capture_format: "markdown"`, `transport: "vscode-claude-code"` — clean self-identification via untyped envelope fields, no `x.<domain>.*` namespace pressure
- `metadata.redaction_status: "raw"`, `metadata.retention_policy: "permanent"`, `metadata.source_jsonl` — extension via metadata, in line with the catdef-family pattern
- Filename pattern `YYYY-MM-DD-HHMM--<role-id>--<subject-slug>.{openthing,body.md}` — single role-id (no double-id pair), matches intra-position single-participant shape

What the implementation surfaced (folded into this revision):

1. **Folder-root anchoring was ambiguous in the original draft.** ccc-ninja anchored at the GitHub-repo root. The revision adds explicit working-repo-root language to the Folder convention section and a new OQ9. Without this clarification, every capture-tool implementer would face the same ambiguity.
2. **`ended` semantics needed tightening.** ccc-ninja populated `ended` mid-session (at first-capture timestamp, while the conversation continued). This is consistent with one reading of the original text ("update once on session close") if you interpret "close" loosely, but conflicts with the append-mode-is-load-bearing rationale: if `ended` mutates on every capture, the envelope is no longer set-once. The revision tightens to "SHOULD be ABSENT while ongoing; populated exactly once on detected close". OQ3 resolved.
3. **Subject derivation grabbed harness noise.** ccc-ninja's subject was derived from the `<ide_opened_file>` system tag rather than the user's first substantive turn ("Hi, could you read /projects/memodef..."). The slug propagated into the filename, degrading discoverability. The revision adds capture-tool quality guidance (CONTRIBUTING-level, not SCHEMA MUST) in the Capture-tool integration section.
4. **`redaction_status` enum confirmed sufficient.** ccc-ninja chose `"raw"` cleanly, exercising one of the three documented values. OQ5 resolved.

What the implementation deliberately did NOT do (out of scope for spec-side; reserved for the parallel ccc-ninja-engineer feedback track):

- Subject anchoring heuristics (which user turn to pick) — capture-tool internal
- Append vs regenerate strategy for the sibling `.body.md` as JSONL grows — capture-tool internal (both are conformant per "regenerate or append" language in the Transcripts folder section)
- Whether to emit an envelope at all on transcripts users haven't explicitly saved — capture-tool UX, not spec shape

These items are queued for direct feedback to the ccc-ninja-engineer seat in the parallel work track (see Director's 2026-05-20 split: "1) spec updates, 2) ccc-ninja-engineer suggestions").

### 0.13.0 conformance confirmed (same day)

ccc-ninja-engineer received the feedback memo (`ccc-ninja/memos/inbox/2026-05-20-1955--memodef-strategist--ccc-ninja-engineer--v0-4-transcript-emission-feedback.openthing`) and shipped **ccc-ninja@0.13.0** the same day, implementing all three refinements. Verified at `memodef/transcripts/memodef-strategist/2026-05-19-1721--memodef-strategist--hey-claude-i-d-like-to-work-on-caliper.openthing` (a back-fill capture of a 2026-05-19 caliper session, written under the new version after VS Code reload):

| Refinement | 0.12.0 behavior | 0.13.0 behavior | Status |
|---|---|---|---|
| Subject derivation | Grabbed `<ide_opened_file>` harness tag | `"Hey Claude -- I'd like to work on Caliper."` — first substantive user turn | ✅ |
| `ended` field | Populated mid-session, advancing on re-save | Field ABSENT from envelope entirely | ✅ |
| Folder-root anchoring | `s:/projects/memodef-spec/transcripts/` (outside git tree — git root is `memodef/`) | `s:/projects/memodef-spec/memodef/transcripts/` (inside git tree, working-repo root) | ✅ |

The folder-root refinement turned out to be even more material than originally framed: the 0.12.0 placement was not just "non-conformant placement within the GitHub repo" but "outside the git-tracked tree entirely" (git root in this project is at `memodef-spec/memodef/`, not `memodef-spec/`). The 1932 transcript pair captured by 0.12.0 was therefore untracked and ungittable. After 0.13.0 conformance was confirmed, the pre-refinement 1932 pair was relocated into the working-repo-root location (now at `memodef/transcripts/memodef-strategist/2026-05-20-1932--...`) to bring the historical artifact into git-tree-conformance — its contents preserved verbatim (including the harness-tag subject) as authentic evidence of pre-refinement output.

Bonus refinement, unsolicited: 0.13.0 emits `metadata.source_jsonl` with forward-slash path separators (`c:/Users/...`) instead of escaped backslashes (`c:\\Users\\...`). Cosmetic but platform-portable; not in the feedback memo's three asks.

All six load-bearing-precedent items preserved across 0.12.0 → 0.13.0 (type stamp, participants single-entry shape, body_ref + no body, untyped first-class envelope fields, metadata extension, single-role-id filename pattern). Version stamp `memodef: "0.4.0"` maintained — appropriate while the proposal awaits Director ratification, since the implementation uses v0.4 features and the version-stamp-consistent-with-features-used rule applies regardless of ratification timing.

**Net effect on proposal status:** v0.4 enters Director ratification as **spec-text-and-implementation-aligned** — proposal text reflects empirical reality from a real first implementation, AND that first implementation is conformant against the proposal's refined text. The capture-loop and the implementation-loop both closed on the same day.

## Cross-references

- v0.3 memos-to-file decision (sentinel-and-folder precedent, POP-discipline operational test): [decisions/proposal-2026-05-10-memos-to-file-and-notes-folder.md](../decisions/proposal-2026-05-10-memos-to-file-and-notes-folder.md)
- v0.2 body_ref decision (sibling .body.md pattern, atomic-move helper): [decisions/proposal-2026-05-01-body-ref-v0.2.md](../decisions/proposal-2026-05-01-body-ref-v0.2.md)
- v0.3.1 implementer-experience clarifications (SHOULD-violation surfacing latitude, hosted-store metadata passthrough): [decisions/v0.3.1-implementer-experience-clarifications.md](../decisions/v0.3.1-implementer-experience-clarifications.md)
- Charter (AI-legibility primacy, POP-discipline, no-vendor-advocacy values): [org/memodef-spec-organization.opencatalog](../org/memodef-spec-organization.opencatalog)
- Architecture-validation decision (folder-as-state, README-vs-SCHEMA placement precedent): [decisions/proposal-2026-04-29-catdef-strategist-architecture-validation.md](../decisions/proposal-2026-04-29-catdef-strategist-architecture-validation.md)
- ccc-ninja capture tool (reference implementation in flight; Director-driven envelope-emission work separate from this proposal): https://github.com/scottconfusedgorilla/ccc-ninja
- First-implementation capture (this proposal's revision session, captured by ccc-ninja@0.12.0): `transcripts/memodef-strategist/2026-05-20-1932--memodef-strategist--ide-opened-file-the-user-opened-the-file-untitled.{openthing,body.md}` (relative to the memodef working-repo root)
- openbraid-engineer v0.3 implementation-experience precedent (first-reader experience folded back into v0.3.1): [decisions/v0.3.1-implementer-experience-clarifications.md](../decisions/v0.3.1-implementer-experience-clarifications.md)
- openbraid hosted-store (potential transport for openbraid-mediated transcripts; API scoping is openbraid's call per Q7): https://openbraid.app
