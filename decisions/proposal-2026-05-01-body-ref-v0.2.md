# Decision — proposal-2026-05-01-body-ref-v0.2

**Disposition:** Accept the proposal as filed. Promote the `body_ref` OPTIONAL field to v0.2.0 of the memodef SCHEMA. `body` remains MUST-non-empty and SHOULD become a 1–3-sentence triage summary when `body_ref` is present. Sibling file co-located as `<memo-filename-without-.openthing>.body.md`. Strictly additive; v0.1 memos remain conformant under v0.2 with no changes. Minor version bump 0.1.0 → 0.2.0.
**Origin:** Strategist-pending [proposals/2026-05-01-body-ref-v0.2.md](../proposals/2026-05-01-body-ref-v0.2.md), filed in response to trigger memo [memos/read/2026-05-01-1430--orgdef-strategist--memodef-strategist--body-ref-case-2-trigger.openthing](../memos/read/2026-05-01-1430--orgdef-strategist--memodef-strategist--body-ref-case-2-trigger.openthing).
**Decided:** 2026-05-01 by memodef-strategist (currently played by the catdef-family bootstrap session arc per [decisions/bootstrap-deviation.md](bootstrap-deviation.md)), Director-ratified same day.
**Validator:** N/A at decision time — schema text drafting follows this artifact (see Workflow validation below). Inbound trigger memo schema-validated as Pass with no notes.

## Source-project peer review

The trigger memo (`from: orgdef-strategist`, `to: memodef-strategist`, sent 2026-05-01T14:30:00-04:00) is the cross-spec coordination signal that fired the body_ref v0.2 trigger. The memo cleanly stayed in scope: surfaced the trigger, recommended promotion to a filed proposal, flagged two design questions for memodef-strategist disposition, and explicitly disclaimed making any of the strategist-scope calls itself. Same-head provenance flagged in the memo's metadata — the orgdef-strategist seat is also held by the bootstrap session arc; the artifact trail nonetheless goes through proper repos so future ratification can audit the cross-spec coordination via committed artifacts.

The orgdef-side coordination proposal ([orgdef-spec/orgdef/proposals/inter-position-communication-convention.md](https://github.com/orgdef-spec/orgdef/blob/main/proposals/inter-position-communication-convention.md)) lands the placement-convention call (memos/ as universal inter-position channel; reject bespoke `handoffs/` category) independently of memodef's body_ref work. The two specs coordinate without coupling.

## Detailed disposition

### Q1 — Body required or optional when `body_ref` is present?

**`body` remains MUST-non-empty. SHOULD become a 1–3-sentence triage summary when `body_ref` is present.**

The alternative (`body` MAY be absent, full defer to `body_ref`) was rejected. AI-legibility primacy ([org/memodef-spec.openthing values](../org/memodef-spec.openthing)) ranks AI-to-AI legibility above human convenience and above auditability. The memo envelope is the load-bearing AI-triage artifact; allowing `body` to be empty would force every recipient to load the sibling file just to triage. That's a per-reader-per-read tax for one-time author convenience — wrong direction on the value hierarchy.

Reframe: `body` is **always** the AI-triage abstract; `body_ref` is the long-form expansion. Author writes both (one-time cost); every reader gets cheap triage forever.

### Q2 — Path convention for the sibling file?

**Co-located `<memo-filename-without-.openthing>.body.md` in the same directory as the memo.**

The alternative (separate `bodies/` subdirectory) was rejected. It breaks the maildir folder-as-state composability that the architecture-validation arc canonized ([decisions/proposal-2026-04-29-catdef-strategist-architecture-validation.md](proposal-2026-04-29-catdef-strategist-architecture-validation.md)). Co-location keeps the file pair atomic in the receiver's mental model, on disk, and in maildir lifecycle moves. The "directory pollution" objection is moot — `inbox/` directories are designed to be browsed, and a paired `.body.md` next to a `.openthing` is the natural shape, not pollution. The atomic-move requirement is mechanically trivial (a 3-line shell helper or a 2-line tool wrapper).

### Open questions from the proposal

1. **Field name `body_ref` vs alternatives** (`body_path`, `body_file`, `external_body`). **`body_ref` adopted.** Matches the catdef-family coordinate-noun convention (`derived_from.url`, `recommended_roles[].role`) — neutral on transport (file or future URL), unambiguous, short.
2. **Co-location SHOULD vs MUST.** **SHOULD.** Consistent with the Inbox lifecycle convention's RECOMMENDED level. Allows adopters whose tooling expects different shapes (e.g., a `bodies/` subdirectory under their org's local convention) to deviate, at the cost of breaking maildir composability for their own setup. The composability cost is theirs to bear.
3. **Validator existence-check level.** **SHOULD (Pass-with-notes on broken reference).** MUST/Fail would couple validators to filesystem state in a way that complicates pure-syntax validation. Validators with filesystem access SHOULD verify; senders are responsible for ensuring the sibling file is committed alongside the memo.
4. **Atomic-move helper script.** **Leave to adopters.** A non-normative note in the README.md helper section documents the pattern. Committing a reference shell helper alongside the spec creates infrastructure-vs-substrate scope-creep; adopters write their own (3-line shell function or 2-line tool wrapper) trivially.
5. **Forward direction `attachments_ref`.** **Out of scope for v0.2; flagged in Items not incorporated.** The single-body extension is the immediate trigger-driven need; multi-attachment patterns can wait for empirical evidence of the right shape.

## Rationale

Two values in the [memodef-spec charter](../org/memodef-spec.openthing) anchor this disposition:

- **AI-legibility primacy** — drives Q1 toward "summary required." When AI-legibility (envelope-as-triage-artifact) trades against author ergonomic (write once vs write twice), AI-legibility wins. The triage tax is paid by every reader on every read; the summary cost is paid by the author once.
- **POP-discipline** — the proposal's strict additivity, optional new field, no required infrastructure, and "leave atomic-move helper to adopters" disposition all flow from worse-is-better. A proposal that added a new artifact category, required new tooling, or coupled the spec to a particular helper implementation would have been the wrong shape regardless of the trigger's urgency.

Maildir composability ([Inbox lifecycle](../README.md#inbox-lifecycle-recommended)) drove Q2: file pairs that move atomically with a single conventional `git mv` (modulo a 3-line wrapper) preserve the property that recipient state lives in the filesystem layout, readable by `ls`. A `bodies/` subdirectory would have split that property in ways the architecture-validation arc explicitly rejected.

## Notable design choices

- **`body` SHOULD be a summary when `body_ref` is present, not MUST.** A memo author whose full content fits comfortably in `body` and who uses `body_ref` only as a redundant pointer to a copy-elsewhere has no requirement to summarize. The summary expectation is anchored in the typical case (long content in body_ref, short summary in body) but doesn't constrain edge cases.
- **`body` remains MUST-non-empty regardless of body_ref presence.** Ensures the no-NULL-context red line ([charter](../org/memodef-spec.openthing) red_lines) is preserved — no path leads to an envelope with empty content.
- **Bare-filename SHOULD on body_ref path.** Path traversal (`../`, `subdir/`) breaks the co-location property and complicates maildir lifecycle. SHOULD-level discourages without forbidding; adopters with non-maildir tooling MAY deviate.
- **Legacy `x.memo.*` form gets parallel `x.memo.body_ref` extension.** Same field semantics under both forms; the typed-vs-legacy migration question is parked separately ([decisions/proposal-2026-04-29-catdef-strategist-architecture-validation.md notes](proposal-2026-04-29-catdef-strategist-architecture-validation.md)).
- **Strategist/maintainer scope-split during bootstrap deferred.** The Director explicitly authorized this seat to implement the spec text given the minor (additive) nature of the change. Decision artifact and SCHEMA.md/README.md edits land in two distinct commits to preserve the strategist/maintainer hat-distinction in git history, even though the same session arc plays both roles per [decisions/bootstrap-deviation.md](bootstrap-deviation.md).

## Items not incorporated

- **`attachments_ref` (multi-file extension of body_ref pattern).** Out of scope for v0.2. If multi-attachment patterns surface in real usage, file as a separate v0.3+ proposal with empirical motivation.
- **Reference shell helper script alongside the spec.** Out of scope per Q4 above. Adopters write their own 3-line helper trivially; the spec doesn't ship tooling.
- **Bumping all SCHEMA.md examples to `"memodef": "0.2.0"`.** Existing v0.1 examples remain at `"memodef": "0.1.0"`. They're correct under v0.2 (rule: "version stamp consistent with features used"); rewriting them to 0.2.0 would imply "must be 0.2.0" which isn't what additivity means. New body_ref example uses `"memodef": "0.2.0"`.
- **`body_ref` MUST-existence (filesystem-state validation level).** Out per Q3; SHOULD-level chosen.

## Workflow validation

- The proposal → decision → implementation flow per CONTRIBUTING.md §3 ran cleanly: trigger memo (2026-05-01) → proposal filed same day (commit `49d4640`) → decision filed (this artifact) → implementation in same session arc (Director-authorized scope expansion).
- Cross-spec coordination via memo + parallel orgdef-side proposal worked as designed: two specs coordinate via committed artifacts, neither couples to the other's timing. Same-head provenance does not undermine the artifact-trail discipline.
- The receiver-commits pattern for incoming peer memos continues to be the de-facto convention in catdef-family (now 3+ data points). One open workflow question is converging.

## Forward-reference resolution

- **memodef-maintainer roledef** (forthcoming) — when authored, MUST inherit the additive-only invariant and the AI-legibility primacy lens established by this disposition. The decision rationale is the canonical reference for "why summary is required when body_ref is present" if future strategist sessions reconsider.
- **memodef-notify spec (Known Work Item)** — when authored, MUST treat body_ref-using memos as a single transport unit (memo + sibling). Notification-layer protocols MUST NOT split a body_ref memo from its sibling file across separate transport events.
- **`attachments_ref` v0.3+ candidate** — this disposition's rationale on Q5 (out of scope for v0.2) flags that multi-attachment patterns wait for empirical motivation. If filed as a future proposal, the cross-reference is here.
- **Conformance harness (Known Work Item, deferred to v0.2+)** — fixtures added in this implementation (canonical pair, regression guard, two invalid counterexamples) are the seed set for the future runtime-aware harness.

## Notes

- **Director-authorized scope expansion.** The strategist and maintainer roles are separate seats per [CLAUDE.md](../CLAUDE.md). For this minor additive change, Director explicitly authorized this seat to implement the SCHEMA text in addition to filing the decision. Future spec changes default to the strategist/maintainer hat-split; this is not precedent for the strategist routinely drafting spec text.
- **`body_ref` was on the strategist's persistent v0.2 forward-work-items list before the trigger fired.** The threshold logic ("when 2+ humans hand-author memos and the pain becomes operationally relevant") was met by case 2 (Thingalog product-strategist, 2026-05-01). Case 1 was brother-canonical-implementor's first memo round-trip (2026-04-27). The trigger logic functioned as designed.
- **Status of conformance/ pre-decision: empty.** Fixtures committed by this implementation are the first conformance fixtures in the spec. Subsequent fixtures will follow this format precedent.

## Cross-references

- Trigger memo: [memos/read/2026-05-01-1430--orgdef-strategist--memodef-strategist--body-ref-case-2-trigger.openthing](../memos/read/2026-05-01-1430--orgdef-strategist--memodef-strategist--body-ref-case-2-trigger.openthing)
- Strategist-pending tracking artifact: [proposals/2026-05-01-body-ref-v0.2.md](../proposals/2026-05-01-body-ref-v0.2.md)
- orgdef-side coordination proposal: [orgdef-spec/orgdef/proposals/inter-position-communication-convention.md](https://github.com/orgdef-spec/orgdef/blob/main/proposals/inter-position-communication-convention.md)
- Charter (AI-legibility primacy value, no-NULL-context red line, no-required-infrastructure red line): [org/memodef-spec.openthing](../org/memodef-spec.openthing)
- Architecture-validation decision (folder-as-state, POP-discipline): [decisions/proposal-2026-04-29-catdef-strategist-architecture-validation.md](proposal-2026-04-29-catdef-strategist-architecture-validation.md)
- Bootstrap deviation context: [decisions/bootstrap-deviation.md](bootstrap-deviation.md)
- Implementation: SCHEMA.md (`body` amendment, `body_ref` SHOULD field, internal consistency rule, legacy mapping row, version bump to 0.2.0); README.md (OPTIONAL list update, atomic-move helper note, roadmap mark); conformance/{valid_memos,invalid_memos}/ (fixtures)
