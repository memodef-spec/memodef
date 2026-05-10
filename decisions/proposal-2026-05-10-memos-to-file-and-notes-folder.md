# Decision — proposal-2026-05-10-memos-to-file-and-notes-folder

**Disposition:** Accept the proposal as filed. Promote `to: "file"` sentinel and `notes/<role-id>/` folder convention to v0.3.0 of the memodef SCHEMA + README. Strictly additive; v0.2 memos remain conformant under v0.3 with no changes. Minor version bump 0.2.0 → 0.3.0. All six open questions resolved per strategist leans (see Detailed disposition).
**Origin:** Strategist-pending [proposals/2026-05-10-memos-to-file-and-notes-folder.md](../proposals/2026-05-10-memos-to-file-and-notes-folder.md), filed in response to the Director's two-axis use-case framing surfaced in conversation 2026-05-02 (cross-runtime + cross-session role-portable context, openbraid as the hosted-store transport-layer implementation in flight).
**Decided:** 2026-05-10 by memodef-strategist (currently played by the catdef-family bootstrap session arc per [decisions/bootstrap-deviation.md](bootstrap-deviation.md)), Director-ratified same day on the laptop arc unblocking openbraid Phase B work.
**Validator:** N/A at decision time — schema text drafting follows this artifact (see Workflow validation below). The proposal artifact itself was schema-validated as Pass during filing review.

## Source-project peer review

This proposal originated in design conversation between Director and memodef-strategist (desktop arc) 2026-05-02, after the Director introduced openbraid as the hosted memo-store transport-layer implementation. The Director's load-bearing observation is the recordable peer-review evidence:

> I may be talking to "an agent who is acting as catdef-strategist", but in reality that is 5 different agents on 2 different platforms over several days, but the *context* is preserved for me because they have written memos-to-file.

Two key validations from the conversation:

1. **POP-discipline course-correction.** The strategist's first sketch proposed a new `memodef:Note` top-level type with role-knowledge-shaped fields. The Director course-corrected toward sentinel-and-folder ("memo to file" from legal practice — `to: "file"` + `notes/<role-id>/`). The course-correction is recorded as the canonical example of POP-discipline in action: when adding to the spec, route to optional fields, sentinels, x.* extensions, or skip — NOT new top-level types. The architecture-validation arc's "SCHEMA reserved for memo SHAPE; folder layout is workflow" rule generalizes: lifecycle differences are folder-convention concerns, not schema concerns.

2. **Operational evidence already exists.** The Director independently tested cross-runtime portability (Claude Code + Claude Desktop + Claude-on-iPhone + Perplexity) with openbraid as the substrate. The proposal's load-bearing claim — that role-portable context across runtimes is achievable through memo-shape primitives — is empirically confirmed before the proposal landed. The spec change is backfilling formalization to a working substrate, not speculating about future use.

The cross-spec coordination posture is clean: openbraid Phase B is gated on this proposal's ratification (per the engineer's note). The spec lands first; the openbraid API surface follows. Same pattern as the orgdef-side `inter-position-communication-convention` proposal landed independently of body_ref v0.2.

## Detailed disposition

### Q1 — Sentinel value `"file"` accepted, or prefer alternatives?

**`"file"` accepted as proposed.** Matches the lawyer mental model anchoring this proposal ("memo to file"). Short. Coordinate-noun-shaped (parallel to existing `"all"`). Alternative candidates (`"record"`, `"log"`, `"memo-to-file"`) read as "what kind of artifact" rather than "where is it filed" — they describe the memo's category instead of its destination, which loses the critical writer's-intent semantic.

### Q2 — `metadata.sender_session_arc` SHOULD vs MUST for memos-to-file?

**Stay SHOULD with strong emphasis** (the proposal's lean). MUST validation can't enforce that the value is meaningful (a session_arc identifier is opaque — the validator can't tell if "claude-opus-4-7-1m, desktop arc 2026-05-02" or any string the author chose is a real or distinguishing identifier). The rule is operational discipline, not schema-enforceable behavior. SHOULD with the internal-consistency-rule emphasis ("load-bearing for distinguishing successive incumbents who share `from`") is the right level — strong enough that conformant memos will populate it, lenient enough not to hard-fail validation on a value-quality call validators can't make.

### Q3 — Per-job notes folder (`notes/<role-id>/<job-id>/`)?

**Out of scope for v0.3.** The role-level notes folder is the primary use case the Director's evidence confirms. Job-specific accumulated context divergence from role-level is hypothetical — defer until empirical evidence surfaces a job whose notes meaningfully diverge from its parent role's. If filed as a future proposal, the cross-reference is here.

### Q4 — Cross-role discoverability?

**Out of scope for v0.3.** Notes are role-scoped by design — a future incumbent of role X reads `notes/X/`, not `notes/Y/`. Cross-role wisdom flows via memos to specific positions, the same as today. If a pattern emerges where roles routinely benefit from each other's notes, file as a future proposal (likely shape: a memo-to-file with `metadata.also_relevant_to_roles: [...]` extension, or a shared `notes/family/` folder for catdef-family-wide context — both speculative).

### Q5 — openbraid API scoping (separate `list_notes` vs folder-prefix on `list_inbox`)?

**Cross-spec coordination, not normative for memodef.** The proposal's posture (memodef lands first; openbraid follows) is the right discipline — memodef defines the memo SHAPE and folder convention; openbraid implements transport. Whether openbraid grows folder-scoping on the existing `list_inbox` or adds a parallel `list_notes` is an openbraid-side architectural call, separate from memodef ratification. Flag for tracking, not blocking.

The strategist's lean — for what it's worth as input rather than directive — is folder-prefix scoping on existing `list_inbox` (e.g., `list_inbox(folder="notes/memodef-strategist/")`) rather than a parallel `list_notes`. Reasons: matches the underlying file model (one transport surface, one storage namespace, two folder conventions); preserves the principle that memos and memos-to-file share a SHAPE; minimizes API surface area. But this is openbraid's call.

### Q6 — `x.org.notes_location` adopter override?

**Out of scope for v0.3; orgdef-side coordination if needed.** memodef defines the default (`notes/<role-id>/`); adopters who need to override declare their convention via an orgdef extension (parallel to the existing `x.org.memo_location` convention for inbox placement). If orgdef-strategist sees adopter pushback on the default, file an orgdef-side proposal to formalize `x.org.notes_location`. memodef itself does not need to enumerate every override mechanism orgdef may declare.

## Rationale

Three values in the [memodef-spec charter](../org/memodef-spec-organization.openthing) anchor this disposition, and a fourth from operational practice:

- **AI-legibility primacy.** A future Claude session reading `notes/memodef-strategist/2026-05-10-1430--alice--file--design-decision.openthing` knows from filename + folder placement: subject, author, role-folder placement, version. Same legibility property as memos. Folder-as-role-context-scope extends the existing folder-as-lifecycle-state pattern (architecture-validation decision); no new field invents what folder placement already encodes.

- **POP-discipline.** Two small additions (sentinel value + folder convention) instead of a new top-level type with parallel API surface. The architecture-validation decision's "SCHEMA reserved for memo SHAPE; folder layout is workflow" rule generalized cleanly: lifecycle differences (flat vs maildir) are workflow concerns. The proposal's first sketch — a new `memodef:Note` type — would have violated POP-discipline by adding a parallel API surface, validator complexity, and conformance-fixture surface area for a use case that the existing memo SHAPE already supports.

- **No vendor advocacy / equal-citizen rule.** The proposal's openbraid cross-reference names openbraid as the hosted-store transport-layer implementation in flight, but the spec itself is transport-agnostic. `notes/<role-id>/` works in git+filesystem (the substrate v0.1 was built on); the same SHAPE works in openbraid (the substrate v0.3 unblocks); the same SHAPE works in any future transport that respects memodef artifacts. Naming openbraid in adopter-facing material is documenting the substrate, not preferring a vendor — the exact same posture established for Claude Code Channels in the architecture-validation decision.

- **Operational evidence beats speculation.** The Director's cross-runtime testing (Claude Code + Desktop + iPhone + Perplexity) before the spec change validated that role-portable context is achievable through memo-shape primitives. The proposal is formalization, not speculation.

## Notable design choices

- **`from` stays the position id, not the session id.** Memos-to-file are authored under role authority, not session authority; successive incumbents share `from = <role-id>` and inherit each other's notes under the same identity. `metadata.sender_session_arc` carries the per-session differentiator. This preserves "the authority of the role" as the durable git-loggable property.

- **Folder placement carries role-scope semantics, not a new field.** Two options were considered — folder-only (this proposal) versus an explicit `role` or `metadata.filed_under_role` field. Folder-only chosen for POP-discipline + simplicity; misplaced files are workflow errors, not schema-validity errors. Extends the architecture-validation precedent that folder placement carries lifecycle semantics.

- **`to: "file"` is a writer's-intent declaration, not a routing instruction.** A memo with `to: "file"` is filed for the role's record by author intent. The folder placement (`notes/<role-id>/`) carries which role's record. The sentinel and the folder are independent dimensions: schema (`to: "file"`) declares the memo's nature; convention (`notes/<role-id>/`) declares the destination.

- **No mark-as-read for memos-to-file.** Lifecycle is "append-only via commit history" (or, in hosted-store transports like openbraid, append-only log). Every successor incumbent reads everything they want to inherit; there's no per-recipient processing event to mark. The receiver-commits convention from `decisions/receiver-commits-convention.md` still applies — the author commits the memo-to-file as receipt-of-record-by-self, and pushes; future readers pull and inherit.

- **Strategist/maintainer scope-split implementation.** Decision artifact authored by `memodef-strategist`; SCHEMA.md / README.md edits + conformance fixtures authored by `memodef-maintainer`. Same Director-authorized scope expansion precedent as body_ref v0.2 — a minor (additive) change handled by one session arc playing both roles, with hat-distinction preserved in commit authorship per [CLAUDE.md:58](../CLAUDE.md).

## Items not incorporated

- **Per-job notes folder** (`notes/<role-id>/<job-id>/`). Out per Q3.
- **Cross-role discoverability mechanism.** Out per Q4.
- **`x.org.notes_location` extension.** Out per Q6 (orgdef-side coordination if needed).
- **Bumping all SCHEMA.md examples to `"memodef": "0.3.0"`.** Existing v0.1/v0.2 examples remain at their version stamps; they're correct under v0.3 (rule: "version stamp consistent with features used"). New memo-to-file example uses `"memodef": "0.3.0"` since it uses the new sentinel.
- **Reference shell helper for memo-to-file authoring.** No spec-shipped tooling; adopters implement trivially (the memo file format is unchanged, just the `to` value and folder placement differ).
- **`memodef:Note` new top-level type** (the original sketch). Rejected per Source-project peer review and Rationale above. The sentinel-and-folder approach achieves the same use case with an order of magnitude less surface area.

## Workflow validation

- The proposal → decision → implementation flow per CONTRIBUTING.md §3 ran cleanly: design conversation 2026-05-02 (Director course-correction recorded in strategist memory) → proposal filed 2026-05-10 (commit `315e88f`) → decision filed (this artifact) → implementation in same session arc (Director-authorized scope expansion + openbraid Phase B unblock motivation).
- Cross-spec coordination via openbraid engineer's note re: Phase B gating worked as designed: external implementation arc reading the proposal directly identified blocking ratification, surfaced through Director, ratification proceeded. Validates the proposal-public-then-ratify pattern.
- The receiver-commits convention from `decisions/receiver-commits-convention.md` continues to be the de-facto operational pattern in this session arc; no friction with v0.3 work.

## Forward-reference resolution

- **memodef-strategist roledef** (forthcoming) — when authored, MUST inherit the POP-discipline-applied-to-memo-shape lens established by this disposition's first-sketch course-correction. The new-type-vs-sentinel-and-folder decision is the canonical example of "is this shape, or is this lifecycle?" Future strategist sessions facing similar enrichment proposals should recognize the pattern.
- **memodef-maintainer roledef** (forthcoming) — MUST inherit the additive-only invariant and the version-stamp-consistent-with-features-used rule clarified by this and body_ref v0.2.
- **openbraid v0 architecture** — the API scoping question (Q5) is now openbraid's call, with strategist input flagged in disposition. Cross-spec coordination memo to openbraid engineer / future openbraid-strategist if/when that seat is provisioned.
- **`attachments_ref` v0.3+ candidate** — the body_ref v0.2 disposition's forward-pointer remains. Memos-to-file using body_ref are particularly likely to surface multi-file body needs (e.g., a memo-to-file documenting a complex design decision with three sibling appendices), strengthening the empirical case for `attachments_ref` if those patterns surface.
- **Cross-role discoverability** (Q4) — the deferred case. If patterns emerge, file as future proposal.

## Notes

- **Director-authorized scope expansion.** Same posture as body_ref v0.2: this decision artifact is filed under memodef-strategist authorship; the implementation (SCHEMA, README, conformance fixtures) is filed under memodef-maintainer authorship in a separate commit. The Director's "ratify and push" instruction includes the implementation given the additive nature of the change and the openbraid Phase B unblock motivation.
- **POP-discipline operational test passed.** This proposal arc's first sketch was an over-engineered new-type proposal that the Director (correctly) course-corrected. The strategist seat self-corrected and filed the right-shaped proposal in the same conversation. The pattern (strategist proposes; Director course-corrects; strategist refiles) is healthy. Future sessions should not assume strategist-first-sketches are right; the POP-discipline lens needs continuous application.
- **Bootstrap-arc strategist context.** Filed by the catdef-family bootstrap session arc playing memodef-strategist informally per `bootstrap-deviation.md`. When the memodef-strategist roledef and seat are formally provisioned, the new strategist may revisit this disposition if real-world adoption surfaces gaps.

## Cross-references

- Strategist-pending tracking artifact: [proposals/2026-05-10-memos-to-file-and-notes-folder.md](../proposals/2026-05-10-memos-to-file-and-notes-folder.md)
- Charter (AI-legibility primacy, POP-discipline, no-vendor-advocacy values): [org/memodef-spec-organization.openthing](../org/memodef-spec-organization.openthing)
- Architecture-validation decision (folder-as-state, POP-discipline canonization, README-vs-SCHEMA placement precedent): [decisions/proposal-2026-04-29-catdef-strategist-architecture-validation.md](proposal-2026-04-29-catdef-strategist-architecture-validation.md)
- body_ref v0.2 decision (additivity precedent, design-rationale-in-decision pattern, AI-legibility primacy applied): [decisions/proposal-2026-05-01-body-ref-v0.2.md](proposal-2026-05-01-body-ref-v0.2.md)
- Receiver-commits decision (workflow convention canonization, applies to memo-to-file authoring): [decisions/receiver-commits-convention.md](receiver-commits-convention.md)
- Bootstrap deviation context: [decisions/bootstrap-deviation.md](bootstrap-deviation.md)
- Implementation: SCHEMA.md (`to` field amendment for `"file"` sentinel, internal consistency rule, legacy mapping row, version bump to 0.3.0); README.md (Notes folder section, repository layout update, two-axis framing); conformance/{valid_memos,invalid_memos}/ (fixtures)
- openbraid Phase B unblock dependency: external — openbraid engineer's note 2026-05-10 surfaced through Director identified this proposal's pending ratification as Phase B blocker
