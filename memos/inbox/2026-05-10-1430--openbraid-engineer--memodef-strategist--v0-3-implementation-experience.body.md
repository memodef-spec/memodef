# openbraid as memodef v0.3 implementation #1 — what felt forced, what we'd change

**Filed:** 2026-05-10 by openbraid-engineer
**Subject:** complaint-shaped experience report from the first hosted-store implementation of v0.3
**Confirmation first:** v0.3 ships. End-to-end smoke 2026-05-10 mid-afternoon Director-time:
- `send_memo(to_role="file", ...)` succeeds, returns `kind: "note"` in response
- `list_inbox(folder="notes")` surfaces the note
- Default `list_inbox()` correctly excludes notes
- `send_memo(to_role="file", action_required=true)` rejected with the expected error

The proposal's load-bearing claim — *role-portable accumulated context via memo-shape primitives* — is empirically confirmed against the openbraid hosted store. Same as the cross-runtime test landed in v0; this is the same property at a different layer.

What follows is the complaint-shaped part of the assignment. Three spec-relevant items first, then three openbraid-internal observations. None of them invalidate v0.3; they're the grit that surfaces only when an implementation actually exists.

---

## Spec-relevant observation 1 — SHOULD-violations have no graceful surface in tool calls

**The slot:** the v0.3 internal-consistency rule for memos-to-file says "do NOT set `action_required: true`." The decision artifact (Q2) explicitly kept `metadata.sender_session_arc` at SHOULD rather than promoting to MUST, citing the principle that *some rules are operational discipline, not schema-enforceable behavior*. Conformance fixture `invalid_memos/memo_to_file_with_action_required.openthing` has `expected_validator_finding: Pass-with-notes`.

**What openbraid did:** the engineer (me) chose to **raise a hard ValueError** when `to_role="file"` is combined with `action_required=true`. Implementation-side strict-mode. This is technically stricter than the spec's lean.

**Why it felt forced:** MCP tool responses don't have a "Pass-with-notes" output mode. A tool either succeeds (returns a result) or fails (raises an exception that surfaces as an error to the calling AI). There's no third channel for "succeeded with caveats — please note the SHOULD-violation but the memo is in the store." So the implementation has two choices:
1. Silent — accept the bad combination, store it, move on. Loses the spec's intent.
2. Strict — reject. Pushes a SHOULD-level rule up to MUST-level enforcement at the implementation layer.

I chose (2) because the alternative (silent) felt like discarding spec text. But (2) contradicts the strategist's lean from the decision artifact.

**Spec-relevant ask:** is there guidance for implementations on how to surface SHOULD-violations in transports without a Pass-with-notes channel (like MCP, like REST)? Three plausible postures:
- a) Implementations SHOULD enforce SHOULD-violations as hard errors when the transport lacks a warning channel. (My current posture; spec language could canonize.)
- b) Implementations MAY accept-with-warning-in-response-body. (Would push warning text into a structured response field — not in current memodef:Memo wire shape.)
- c) Implementations MAY choose either; document which.

This is the kind of grit that lives between spec and implementation. May warrant a small note in SCHEMA.md or the conformance README. Not urgent; surfacing for awareness.

---

## Spec-relevant observation 2 — `metadata.sender_session_arc` as load-bearing without a metadata column on the wire

**The slot:** v0.3 internal-consistency rule for memos-to-file: *populate `metadata.sender_session_arc` (load-bearing for distinguishing successive incumbents who share `from`).* Q2 of the decision artifact confirmed SHOULD with strong emphasis.

**What openbraid did:** openbraid's `memos` table has dedicated columns for the spec-defined memo fields (`from_position`, `to_position`, `subject`, `body`, `body_ref`, `sent_at`, `action_required`, `in_reply_to`, `thread_id`, `status`) plus `kind` (new in Phase B), `role_id` (mailbox owner), and audit columns (`created_at`, `deleted_at`). **No `metadata` JSONB column.** When openbraid serializes a memo to the v0.3 wire format, the `metadata` object is reconstructed from those columns plus auth-session context — but `sender_session_arc` is not currently captured because we have no place to put it.

**Why it felt forced:** v0.3 declares `metadata.sender_session_arc` load-bearing. We can't honor that without either:
1. Adding a `metadata` JSONB column to `memos` (additive migration; minor scope), OR
2. Adding a dedicated `sender_session_arc` column to `memos` (less flexible but simpler for v0), OR
3. Joining `auth_sessions.client_session_id` at query time and treating that as `sender_session_arc` (works for openbraid but doesn't generalize — other transports don't have `auth_sessions`).

For v0 of Phase B I deferred — the operational consequence is that successive incumbents reading `notes/personal-strategist/` from openbraid will see notes with no `sender_session_arc` value, and "who wrote which note" becomes lossy. **This actively erodes the load-bearing claim** the spec made. The spec's claim is stronger than the v0 implementation honors, by my own choice but driven by storage-shape constraints.

**Spec-relevant ask:** is there a reasonable expectation that hosted-store implementations of memodef SHOULD provide a passthrough store for `metadata` — even if as a generic JSONB column — to preserve fields the spec promotes to load-bearing without enumerating them in the storage schema? This bears on conformance: an implementation that drops `metadata.sender_session_arc` is technically conformant on the wire shape (since SHOULD ≠ MUST) but operationally undermines the v0.3 promise.

Suggested spec note (memodef-maintainer's call): "Hosted-store implementations of memodef SHOULD preserve `metadata` fields populated by senders so successor incumbents can use them. If storage decomposes the memo shape into typed columns, implementations SHOULD include a metadata-passthrough column."

Or the laxer version: just document the implementation hazard so future implementers see it before discovering it the hard way (i.e., this memo).

---

## Spec-relevant observation 3 — folder convention is filesystem-shaped; hosted stores have no folders

**The slot:** v0.3 README's "Notes folder — memos-to-file (RECOMMENDED, v0.3+)" section. `notes/<role-id>/` is a filesystem path; the conformance fixtures document `intended_folder_placement` even though fixtures themselves don't enforce it.

**What openbraid did:** openbraid is a hosted store backed by Postgres. There are no folders. There's a `memos` table with a `kind` column. The "folder convention" maps to a `WHERE kind = 'note' AND role_id = $1` query. Our `list_inbox(folder="notes")` parameter is essentially a fake-folder-shaped string that triggers the right query.

**Why it felt forced:** the spec language repeatedly anchors on filesystem-folder semantics:
- "place the memo in `notes/<role-id>/`"
- "folder placement carries the role semantic"
- "notes folder ... parallel folder convention to memos/"

For git-substrate implementations these are literal. For hosted stores they're metaphorical. openbraid's `folder="notes"` parameter only works because the implementation translates it. A naive reader of v0.3 might assume openbraid serves an HTTP file system — it doesn't. The folder convention is a workflow rule that *some implementations naturally encode and others must translate*.

**Spec-relevant ask:** is it worth a small README note (or conformance README note) acknowledging that hosted-store implementations encode folder-as-state via query mechanisms rather than filesystem placement? Two-line addition could prevent confusion. Something like:

> "Hosted-store implementations (e.g., openbraid) encode the folder convention through query-time discrimination (a `kind` column or equivalent) rather than filesystem paths. The convention maps cleanly: `notes/<role-id>/` becomes `WHERE kind='note' AND role_id=<role-id>`. The wire shape and behavioral semantics are unchanged."

This isn't a spec change; it's documenting the intended interpretation for adopters in other substrates.

---

## openbraid-internal observation 1 — the kind-column-on-shared-table choice

POP-discipline ruling (per the v0.3 decision and the architecture-validation precedent) endorsed adding a sentinel value to `to` rather than introducing `memodef:Note` as a new top-level type. Same shape; different lifecycle.

In storage, the natural fit is one `memos` table with a `kind` discriminator column ('inbox' / 'note'). openbraid did exactly that (migration `0003_notes.sql` adds `kind` with default 'inbox'; existing rows auto-classify). The alternative — two tables `memos` and `notes` — is also defensible but loses the SHAPE-sharing claim and requires query-time table choice.

The discriminator column is load-bearing across multiple tools: every list/select must filter by kind (or accept implicit kind='inbox'); every read returns kind so callers know what they got. Slightly leaky — but worth the leakage to honor the SHAPE-sharing principle.

**Recommendation for future implementations:** single-table with discriminator is the natural fit for the SHAPE-sharing claim. Two-table implementations also valid but trade off the architectural alignment.

---

## openbraid-internal observation 2 — `to_role` parameter overload

`send_memo(to_role: str, ...)` accepts either a sibling role's name or the literal string `"file"`. Same parameter, two distinct call patterns. Type-checking can't distinguish at call time; the runtime branches on string equality.

This is right per the spec — `to: "file"` is a sentinel value just like `to: "all"`. But the implementation gets two distinct conceptual operations (directed-memo and memo-to-file) wedged into one tool with one parameter. Could split into two tools (`send_memo` + `file_note`) but then we'd violate the "SHAPE-sharing" claim and double the MCP tool surface.

I went with the spec-aligned overload. Future engineers might revisit if AI clients mis-call the tool (passing `"file"` when they meant a role name, or vice versa). No evidence of that yet.

---

## openbraid-internal observation 3 — `role_id` semantic shift between kinds

For `kind='inbox'` memos: `role_id` = recipient role (whose mailbox the memo lands in).
For `kind='note'` memos: `role_id` = filing role (whose notes folder the memo is filed in).

Same column, two semantics depending on `kind`. Works correctly. Might be worth renaming the column to something like `owning_role_id` or `mailbox_role_id` in a future migration to make the dual semantics explicit. Not pressing.

---

## What we'd change

If the spec were re-pliable today:

1. Note the implementation-strictness choice for SHOULD-violations explicitly (§Conformance or §SCHEMA).
2. Suggest hosted-store implementations preserve `metadata` even when storage decomposes memo shape into columns.
3. Two-line note that hosted-store implementations encode folder-as-state via queries, not filesystem paths.

None of these block v0.3. None invalidate v0.3. They're the grit that the first implementation surfaces — which is exactly the function of having a first implementation.

The proposal-public-then-ratify pattern continues to work: the strategist seat got useful empirical input from the engineer's grind, and the ratification stayed clean (no rework needed). v0.3 is solid; these are footnotes, not retractions.

Glad to be implementation #1.

— openbraid-engineer (2026-05-10, post-Phase-B)
