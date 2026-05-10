# Memos-to-file and `notes/<role-id>/` folder convention (v0.3)

**Disposition:** Strategist-pending — filed by memodef-strategist; Director ratification needed before maintainer drafts SCHEMA.md text
**Origin:** Director use case surfaced 2026-05-02: cross-runtime + cross-session role-portable context. Designed in conversation 2026-05-02 after Director course-corrected an over-engineered first sketch (proposed new `memodef:Note` top-level type) toward POP-discipline (sentinel + folder convention only).
**Filed:** 2026-05-10 by memodef-strategist
**Schema impact:** strictly additive; minor version bump 0.2.0 → 0.3.0

## Summary

Add a `to: "file"` sentinel value to `memodef:Memo` (parallel to the existing `to: "all"` broadcast sentinel), and a `notes/<role-id>/` folder convention for filing those memos. Memos-to-file are the **memo-to-file pattern** from legal practice: a memo authored by a role-incumbent, filed for the role's record rather than directed at any recipient. Successive role-incumbents read the accumulated notes to inherit context. Strictly additive — existing inline-recipient and broadcast memos remain valid.

## Motivation

memodef v0.1–v0.2 served **inter-position coordination** well: handoffs across roles (strategist→maintainer→implementer), action requests, threaded replies, broadcasts. The maildir lifecycle (inbox/read/archive) is correct for that case.

A second use case has emerged in operational practice: **intra-position context portability**. A human Director interacts with what feels like one durable role (e.g., catdef-strategist), but behind that role any number of agents play across runtimes (Claude Code, Claude Desktop, Claude-on-iPhone, Perplexity have all been operationally tested) and over multiple days. The Director's mental model is "I'm talking to catdef-strategist" — but the only thing that makes that mental model tell the truth instead of being a polite fiction is **accumulated role context that successive incumbents read on session start**.

Today the workaround is session-handoff memos: a previous incumbent files a memo to the next incumbent in `memos/inbox/`, who reads it, marks-as-read, processes the content. That works for one-off handoffs but degrades for accumulated knowledge: every successor incumbent has to re-read the entire chain of session-handoff memos in `memos/read/` to reconstruct context, the inbox/read distinction becomes meaningless (every memo is "read by some prior incumbent"), and the from/to fields are awkward (`from: catdef-strategist, to: catdef-strategist` for self-handoffs).

The lawyer "memo to file" pattern fits this exactly. A lawyer working a matter writes memos that go into the matter's file — not addressed to anyone, but a record of analysis for whoever next works the matter. Memodef's analog: a role-incumbent writes a memo `to: "file"`, files it in `notes/<role-id>/`, and any future incumbent of that role inherits the accumulated context by reading the folder.

The use case is now operationally bounded by openbraid (hosted memo store, in flight) — the same role-portable context pattern works across runtimes that don't / can't use git+filesystem, with the same memo shape.

## Proposed Change

### Schema delta (SCHEMA.md)

Two amendments to the existing `to` field:

#### `to` (string) — amendment

Current text describes `to` as "the recipient's position id, OR the literal string `\"all\"` for org-wide broadcasts." Add: "OR the literal string `\"file\"` for **memos-to-file** — informational records filed for a role's accumulated context rather than directed at any position."

```json
"to": "file"
```

The role context for a memo-to-file comes from **folder placement**: a memo with `to: "file"` SHOULD live in `notes/<role-id>/<filename>.openthing` where `<role-id>` is the position id whose accumulated context the memo enriches. The folder placement carries the role semantic; the schema does not introduce a new field for it.

`from` for a memo-to-file is the position id of the role-incumbent who authored the note (the same value all successive incumbents share when filing under that role). `metadata.sender_session_arc` becomes load-bearing for memos-to-file as the only differentiator between successive incumbents — see internal consistency rules.

#### Internal consistency rule (SCHEMA.md §Internal consistency rules)

Add to the SHOULD-list: "When `to` is `\"file\"`: place the memo in `notes/<role-id>/` where `<role-id>` is the role whose context is being accumulated; populate `metadata.sender_session_arc` with the authoring session's identifier (load-bearing for distinguishing successive incumbents who share `from`); do NOT use the maildir inbox/read/archive lifecycle (memos-to-file are flat — no per-recipient processing event to mark)."

#### Filename convention (RECOMMENDED, SCHEMA.md §File naming convention)

Existing convention: `YYYY-MM-DD-HHMM--<from>--<to>--<short-subject>.openthing`. Memos-to-file naturally compose: `<to>` becomes `"file"`. Example: `notes/memodef-strategist/2026-05-10-1430--alice--file--why-summary-required-when-body-ref-present.openthing`.

#### Legacy form (SCHEMA.md §Backward-compatible legacy form)

The legacy `x.memo.*` form gets the same sentinel: `"x.memo.to": "file"`. Same shape, same semantics.

### Folder convention (README.md)

Add a new top-level section after **Inbox lifecycle (RECOMMENDED)**:

#### Notes folder — memos-to-file (RECOMMENDED, v0.3+)

A second folder convention parallel to `memos/`, for **role-portable accumulated context**:

- `notes/<role-id>/` — memos filed for a role's record (not directed at any position; flat, no inbox/read/archive lifecycle)
- One folder per role: `notes/memodef-strategist/`, `notes/catdef-strategist/`, etc.
- Lifecycle is "append-only via commit history" (or, in hosted-store transports like openbraid, append-only log)
- No mark-as-read — every successor incumbent reads everything they want to inherit

The two folder conventions serve two axes:

| Axis | Folder | Lifecycle | Use case |
|---|---|---|---|
| Inter-position coordination | `memos/inbox/` → `read/` → `archive/` | Maildir (per-recipient processing) | Handoffs, action requests, replies, broadcasts |
| Intra-position context portability | `notes/<role-id>/` | Flat (append-only) | Role-incumbent accumulated context for successors |

A single repo MAY use both, neither, or just one. RECOMMENDED, not REQUIRED. Adopters whose context patterns diverge from these defaults MAY use other folder layouts.

### Repository layout (README.md repository layout block)

Update the repository layout block to show:

```
memodef/
├── ...
├── memos/                      ← inter-position coordination (existing)
│   ├── inbox/
│   ├── read/
│   └── archive/
├── notes/                      ← intra-position context (new in v0.3)
│   └── <role-id>/
└── ...
```

## Design rationale (for the strategist-scope calls)

### Why `to: "file"` sentinel, not a new top-level type

The first sketch (designed 2026-05-02 by this strategist seat, then rejected after Director course-correction) proposed a new `memodef:Note` top-level type with role-knowledge-shaped fields (no `to`, explicit `role`, etc.). That direction was wrong because:

1. **POP-discipline.** New top-level types are the heaviest enrichment in the spec's vocabulary. Memos-to-file are still memos — same `from`, same `subject`, same `sent`, same `body`, same `body_ref`, same `in_reply_to` (for "Building on alice's earlier note..." threading). The differences are: a new sentinel value for `to`, a new folder convention. Three lines of spec text, not a new type.

2. **MCP / API surface preservation.** Hosted-store implementations like openbraid already have `send_memo` / `list_inbox` / `read_memo` / `mark_read`. A new type would force a parallel `send_note` / `list_notes` / `read_note` API surface (or a `type=memo|note` discriminator everywhere). The sentinel-and-folder approach reuses the existing surface — list-by-folder-prefix is enough to scope to notes.

3. **Schema coherence.** The architecture-validation decision established that "SCHEMA.md is reserved for memo SHAPE; folder layout is workflow." Lifecycle differences (inbox/read/archive vs flat) are workflow concerns, not schema concerns. A different shape would be a new type; a different lifecycle is a new folder convention.

### Why folder placement carries the role semantic, not a new field

Two options for declaring which role's notes a memo-to-file belongs to:

| | Folder-only | Explicit field |
|---|---|---|
| Where role lives | `notes/<role-id>/` | New `role` or `filed_under_role` field, or `metadata.filed_under_role` |
| Cost | Convention only, no schema change | New SHOULD field |
| Risk | Misplaced files lose their role context | Field can disagree with folder placement |

Folder-only chosen for POP-discipline + simplicity. The architecture-validation precedent already established that folder placement carries lifecycle semantics (inbox/read/archive); extending that pattern, folder placement carrying role-scope semantics is consistent. Misplaced files are a workflow error, not a schema-validity error.

### Why `from` stays the position id (not the session id)

A memo-to-file is authored by a session, but the AUTHORITY of the note is the role's accumulated practice — successive incumbents inherit and extend each other's notes under the same role identity. `from: <position-id>` (e.g., `from: "catdef-strategist"`) preserves that authority, even though multiple session arcs share that value.

`metadata.sender_session_arc` (already SHOULD-fill in v0.1) carries the per-session differentiator. For memos-to-file, this becomes load-bearing — a future incumbent reading `notes/catdef-strategist/` needs to know which note came from which incumbent's session arc to correctly attribute and possibly contradict. The internal consistency rule promotes this from soft-SHOULD to a strongly-recommended-for-memos-to-file.

## Backward Compatibility

Strictly additive. No existing `memodef:Memo` artifact becomes invalid. All v0.2 memos remain conformant under v0.3 — `to: <position-id>` and `to: "all"` continue to mean exactly what they meant. The `to: "file"` value is **new** in v0.3; v0.2 readers would not recognize it but per catdef's reader-lenient discipline, an unknown sentinel value is a Pass-with-notes finding rather than a hard fail. Memos using `to: "file"` SHOULD declare `"memodef": "0.3.0"` per the version-stamp-consistent-with-features-used rule.

`memos/inbox/`, `memos/read/`, `memos/archive/` lifecycle is unchanged. The new `notes/<role-id>/` folder is parallel, not a replacement.

The legacy `x.memo.*` form gets parallel sentinel support for `"x.memo.to": "file"` by the same shape.

## Conformance Tests

To add to `conformance/`:

- **`valid_memos/memo_to_file_basic.openthing`** — canonical memo-to-file. `to: "file"`, `from: "memodef-strategist"`, `metadata.sender_session_arc` populated, lives under hypothetical `notes/memodef-strategist/` placement (fixtures don't enforce filesystem placement; fixture documents the intended location).
- **`valid_memos/memo_to_file_with_threading.openthing`** — successor incumbent's memo-to-file with `in_reply_to` referencing a predecessor's memo-to-file. Demonstrates threading composition for accumulated discussion across incumbents.
- **`valid_memos/memo_to_file_with_body_ref.openthing`** — composes with v0.2 `body_ref` for long-form notes. Sibling `.body.md` co-located. Demonstrates the v0.2 ergonomic relief layered on the v0.3 use case (the memos-to-file case is exactly the long-form-hand-authored content body_ref was built for).
- **`invalid_memos/memo_to_file_with_action_required.openthing`** — `to: "file"` combined with `action_required: true`. SHOULD-level violation: a filed-for-record memo cannot have action-required semantics (no recipient to act). Pass-with-notes.

The folder-placement convention (memo-to-file in `notes/<role-id>/` vs `memos/`) is a workflow rule that fixtures can document but not strictly enforce — fixtures live in `conformance/`, not in role-folder positions. Validators with filesystem-state access MAY check that memo-to-file artifacts are in `notes/` folders.

## Alternatives Considered

- **New `memodef:Note` top-level type** (the first sketch). Rejected per Design rationale — POP-discipline + MCP surface preservation + schema coherence argue against. The sentinel-and-folder approach achieves the same use case with an order of magnitude less surface area.
- **Folder convention with `to: <role-id>` (no sentinel).** Considered: a memo-to-file could use `to: "memodef-strategist"` with folder placement at `notes/memodef-strategist/` carrying the "filed not delivered" semantic. Rejected — loses the writer's-intent distinction between "directed to whoever currently holds this role" and "filed for the role's record." Both should be expressible.
- **Sender-only-can-write semantics.** Considered: restrict `to: "file"` to sessions whose `from` matches the role-folder. Rejected as over-prescriptive — a peer-strategist may legitimately file an observation in another role's notes folder ("memodef-strategist note from catdef-strategist about pattern observed across catdef-family"). The folder placement is the convention; cross-role authoring is permitted.
- **Per-job notes folder (`notes/<role-id>/<job-id>/`).** Out of scope for v0.3. If empirical evidence surfaces that job-specific accumulated context diverges from role-level enough to need its own folder, file as a future proposal.
- **`memodef:Library` of notes.** Existing type for catalogs of memos/templates. Not the right shape — `memodef:Library` is for index artifacts (catalog metadata), not for the underlying notes themselves.
- **Renaming "notes" to "journal" / "log" / "kb" / "scratchpad".** Rejected per design conversation — "note" is what people actually call this thing in legal practice ("memo to file") and in operational use; the other terms misframe.
- **Defer to v0.4 or later.** Rejected — the use case is operationally bounded (cross-runtime testing already done with openbraid in flight); deferring forces adopters to invent local conventions that fragment.

## Open Questions

For Director ratification:

1. **Sentinel value `"file"`** accepted, or prefer `"record"` / `"log"` / `"memo-to-file"`? Lean: `"file"` matches the lawyer mental model and is shortest; the others read as "what kind of artifact" rather than "where is it filed."
2. **`metadata.sender_session_arc` strengthening.** This proposal adds an internal-consistency-SHOULD that says "populate it for memos-to-file." Should it instead be promoted from SHOULD to MUST for the memo-to-file case? Lean: keep SHOULD — even MUST validation can't enforce that the value is meaningful (a session ID is opaque), so the rule is operational discipline, not schema-enforceable. Strongly-SHOULD is right.
3. **Per-job notes folder.** Confirm out of scope for v0.3 (this proposal's posture).
4. **Cross-role discoverability.** A note useful to multiple roles is accessible only by reading their folders directly today. Out of scope for v0.3? Lean: yes, defer. Cross-role wisdom can flow via memos to specific positions for now, the same as today.
5. **openbraid API scoping.** This proposal lands first; openbraid API follows. Whether `list_inbox` grows folder-scoping (`list_inbox(folder="notes/memodef-strategist/")`) or a parallel `list_notes` is added is a separate openbraid-side decision, not normative for memodef. Flagged for cross-spec coordination tracking.
6. **`notes/` folder default for adopters.** RECOMMENDED-not-REQUIRED at the spec level (this proposal's posture). Adopters whose orgs declare a different memo-location convention via `x.org.memo_location` MAY also declare a `x.org.notes_location` extension to override. Not in this proposal's scope; flag for orgdef-side coordination if/when adopters push back on the default.

## Cross-references

- Two-axis framing source: design conversation 2026-05-02, captured in memodef-strategist memory file
- Charter (AI-legibility primacy, POP-discipline): [org/memodef-spec-organization.openthing](../org/memodef-spec-organization.openthing)
- Architecture-validation decision (folder-as-state, POP-discipline canonization, README-vs-SCHEMA placement precedent): [decisions/proposal-2026-04-29-catdef-strategist-architecture-validation.md](../decisions/proposal-2026-04-29-catdef-strategist-architecture-validation.md)
- body_ref v0.2 decision (additivity precedent, design-rationale-in-decision pattern, AI-legibility primacy applied): [decisions/proposal-2026-05-01-body-ref-v0.2.md](../decisions/proposal-2026-05-01-body-ref-v0.2.md)
- Receiver-commits decision (workflow convention canonization without preceding proposal artifact): [decisions/receiver-commits-convention.md](../decisions/receiver-commits-convention.md)
- Existing `to: "all"` sentinel (parallel pattern): [SCHEMA.md → `to` field](../SCHEMA.md#to-string)
- Existing maildir Inbox lifecycle (parallel folder convention to extend): [README.md → Inbox lifecycle](../README.md#inbox-lifecycle-recommended)
- openbraid (hosted memo store, in flight as transport-layer implementation): https://openbraid.app
