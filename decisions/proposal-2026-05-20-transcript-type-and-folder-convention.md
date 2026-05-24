# Decision — proposal-2026-05-20-transcript-type-and-folder-convention

**Disposition:** Accept the proposal as filed-and-revised. Promote `memodef:Transcript` top-level type and `transcripts/<role-id>/` folder convention (at working-repo root) to v0.4.0 of the memodef SCHEMA + README. Strictly additive; v0.1/v0.2/v0.3/v0.3.1 memos remain conformant under v0.4 with no changes. Minor version bump 0.3.1 → 0.4.0. All nine open questions resolved per strategist leans (see Detailed disposition).
**Origin:** Strategist-pending [proposals/2026-05-20-transcript-type-and-folder-convention.md](../proposals/2026-05-20-transcript-type-and-folder-convention.md), filed 2026-05-20 (commit `eb9f762`) in response to Director use cases surfaced 2026-05-19 (cross-runtime + cross-session role-portable VERBATIM context, distinct from memos' DISTILLED context). Revised in place 2026-05-20 (commit `d50773e`) folding first-implementation observations from ccc-ninja@0.12.0 capture back into proposal text pre-ratification.
**Decided:** 2026-05-21 by memodef-strategist (currently played by the catdef-family bootstrap session arc per [decisions/bootstrap-deviation.md](bootstrap-deviation.md)), Director-ratified same day.
**Validator:** memodef-strategist self-validation in validator capacity (validator-as-CI-step automation remains a Known Work Item). The proposal artifact was schema-validated as Pass during filing review; the revision was reviewed against first-implementation evidence before this artifact was filed.

## Source-project peer review

This proposal originated in design conversation between Director and memodef-strategist (vscode arc) 2026-05-19/20, after the Director surfaced three use cases for VERBATIM conversation recording: (1) snippets for books and presentations, (2) lost-window recovery, (3) audit-trail pinning of distillations to verbatim source. The strategist's first sketch — a separate sibling `transcriptdef-spec` — was Director-corrected toward "class of memos in memodef" (recorded in proposal Origin section), keeping the artifact family unified rather than fragmenting across specs.

The unique empirical signal for this proposal is the **same-day first-implementation conformance** by ccc-ninja:

> ccc-ninja@0.12.0 began emitting `memodef:Transcript` envelope + sibling `.body.md` pairs ahead of ratification, capturing the very session that drafted this proposal (capture-loop closed on itself). The first capture surfaced three concrete refinements (folder-root anchoring ambiguity, `ended` semantics in append-mode, subject derivation skipping harness tags). All three were folded into the proposal text pre-ratification. ccc-ninja-engineer received a typed feedback memo and shipped @0.13.0 the same day implementing all three refinements cleanly. v0.4 enters ratification spec-text-and-implementation-aligned.

This is the **second application** of the first-implementation-folded-back-pre-ratification pattern (v0.3 → v0.3.1 via openbraid-engineer 2026-05-10 was the first). The pattern's value is empirically reinforced: real implementations surface ambiguities that proposal authors miss, and folding the refinements back BEFORE the decision artifact lands gives ratification a chance to weigh in on text that has been touched by reality.

The cross-spec coordination posture is clean: ccc-ninja is a capture tool, not a sibling spec; engineer feedback was filed as a typed memodef:Memo per the established cross-repo coordination convention. Same pattern as openbraid Phase B coordination for v0.3.

## Detailed disposition

### Q1 — `body` field on `memodef:Transcript`: SHOULD NOT vs MUST NOT

**SHOULD NOT** (the proposal's lean). Adopters who include a body summary aren't actively harmful — they're over-engineering by their own choice. Implementations encountering a `body` field on a `memodef:Transcript` report Pass-with-notes per the v0.3.1 SHOULD-violation latitude. Stricter MUST NOT would FAIL such artifacts, which is harsher than the design intent warrants.

### Q2 — `body_ref` MUST vs SHOULD for `memodef:Transcript`

**MUST** (the proposal's lean). Without `body` and without `body_ref`, the envelope has no content reference. Relaxing to SHOULD would permit envelope-only "stub" transcripts, which serves no use case the three audiences (Director snippet-search, lost-window recovery, audit-trail pinning) need.

### Q3 — `ended` field semantics on close

**SHOULD be ABSENT while the conversation is ongoing or its close cannot be detected; populated exactly once on detected close.** Resolved by ccc-ninja@0.12.0 first-implementation evidence (recorded in proposal "First-implementation observations" section). Mid-session-snapshot population defeats the append-mode-is-load-bearing property — `ended` is the envelope's only mutable field; mutating it on every snapshot conflicts with the "envelope set once, body grows" design rationale. SHOULD level (not MUST) confirmed given close-detection variability across transports (vscode + ccc-ninja requires explicit user action; openbraid can auto-detect; others vary).

### Q4 — Filename pattern

**Current pattern accepted: `YYYY-MM-DD-HHMM--<primary-role-id>--<short-subject>.openthing`** (deviation from memodef:Memo's `<from>--<to>--<subject>` pattern is appropriate — transcripts have no single from/to, so primary-role-id replaces both; the full participants[] list lives inside the envelope). ccc-ninja@0.13.0 exercises this cleanly. Visual-parallelism alternative (`<role-id>--transcript--<subject>`) rejected — the `transcripts/<role-id>/` folder placement already carries the "this is a transcript" signal; adding `--transcript--` to filenames is redundant.

### Q5 — `metadata.redaction_status` enum values

**Three values accepted: `raw | partial | redacted`.** Resolved by ccc-ninja@0.12.0 first-implementation evidence — ccc-ninja chose `"raw"` cleanly. Adopters needing more granularity (e.g., `pre-redaction-review`, `auto-redacted`, `human-reviewed`) extend via `x.<domain>.*`. Default value is `"raw"`.

### Q6 — `related_memos[]` path resolution

**Keep loose (bare filenames OR paths) for v0.4.** The field is informational, not behavior-driving — validators don't dereference or resolve paths against any canonical location. Adopters use whichever shape composes with their working-repo's discovery patterns. If a pattern emerges where ambiguous shapes cause discoverability failures, file as future proposal (likely shape: SHOULD-guidance recommending bare filenames + co-located resolution).

### Q7 — openbraid API scoping

**Cross-spec coordination, not normative for memodef.** Same posture as v0.3 Q5: memodef defines artifact SHAPE; openbraid implements transport. Whether `send_memo` extends to transcripts via `kind="transcript"` discriminator or a parallel `send_transcript` MCP tool is openbraid's architectural call, separate from memodef ratification. Flag for tracking, not blocking.

### Q8 — ccc-ninja integration

**Out of scope for the spec; empirically validated.** memodef defines artifact SHAPE; capture tools (ccc-ninja and others) implement emission. ccc-ninja@0.13.0 demonstrates a conformant implementation exists; subsequent capture tools (openbraid-side capture, future Claude Code native, third-party) are equally valid. The "Capture-tool integration expectations" section in README.md documents the SHOULD-pattern for tools that emit envelope + sibling pairs; tools that emit only markdown produce conformant content but require manual envelope authoring.

### Q9 — Folder-root anchoring (NEW, surfaced by revision)

**Working-repo root.** The `transcripts/` folder lives at the same directory level as `CLAUDE.md`, `SCHEMA.md`, `README.md`, `memos/`, `decisions/`, `org/` — matching the placement of `memos/` and `notes/<role-id>/`. In repos where the working directory is nested inside a wrapping GitHub repo (e.g., `memodef-spec/memodef/` inside the `memodef-spec` GitHub repo), `transcripts/` lives at `memodef-spec/memodef/transcripts/`, NOT `memodef-spec/transcripts/`. ccc-ninja@0.12.0's GitHub-repo-root anchoring surfaced the original ambiguity — and revealed that the "wrong" anchoring placed transcripts entirely OUTSIDE the git-tracked tree in this project's layout, far worse than just being non-standard. ccc-ninja@0.13.0 implements correct working-repo-root detection.

## Rationale

Three values in the [memodef-spec charter](../org/memodef-spec-organization.opencatalog) anchor this disposition, plus a fourth from operational practice:

- **AI-legibility primacy.** A future Claude session reading `transcripts/memodef-strategist/2026-05-19-1721--memodef-strategist--hey-claude-i-d-like-to-work-on-caliper.openthing` knows from filename + folder placement: subject, primary role, working-repo-root scoped, version. Same legibility property as memos and memos-to-file. The three folder conventions (`memos/`, `notes/<role-id>/`, `transcripts/<role-id>/`) compose cleanly under a single pattern: folder placement carries audience-scope semantics; envelope carries content semantics.

- **POP-discipline.** A new top-level type (rather than sentinel-on-memo) was the right call here because `memodef:Transcript` is **structurally different** from `memodef:Memo`: multi-participant (no single `from`/`to`), multi-timestamp (`started` + `ended` rather than single `sent`), append-only growing content (no abstract regeneration friction), no action_required semantics, no threading. Forcing transcripts into memo shape via `to: "transcript"` sentinel would distort five fields' semantics. The POP-discipline lens distinguishes between "shape" (when types differ structurally — new type warranted) and "lifecycle" (when types share structure but differ in workflow — sentinel + folder sufficient). Memos-to-file (v0.3) was lifecycle; transcripts (v0.4) is shape.

- **No vendor advocacy / equal-citizen rule.** ccc-ninja is named in adopter-facing material as a reference implementation in flight, with explicit no-vendor-preference disclaimer. The spec defines SHAPE; capture tools (ccc-ninja, openbraid-side, future native, third-party) are equally valid. Same posture established for Claude Code Channels in architecture-validation, openbraid in v0.3, and now ccc-ninja in v0.4.

- **Operational evidence beats speculation.** The Director's three use cases (book snippets, lost-window recovery, audit-trail pinning) were grounded in real practice before the proposal landed. ccc-ninja@0.13.0's same-day conformance closes the loop — proposal text reflects empirical reality AND that empirical reality is conformant against the proposal. Same precedent as v0.3 (memos-to-file empirically validated by Director's cross-runtime testing before the spec change).

## Notable design choices

- **NEW top-level type, not sentinel.** Per POP-discipline rationale above. Precedent exists: `memodef:Memo` + `memodef:Library` were already two top-level types; `memodef:Transcript` joins the family.

- **No `body` field on `memodef:Transcript` envelopes.** Append-mode operation is the load-bearing property the type was designed around. Capture tools regenerate or append to the sibling `.body.md` as conversations grow; the envelope is set ONCE on transcript creation and the only mutable field is `ended` (set exactly once on detected close, otherwise absent). This is the inverse of v0.2's body_ref design for `memodef:Memo`: there, body was MUST-non-empty as an AI-triage abstract; here, the audience (Director snippets, recovery, citation; AI context-rebuild) doesn't need triage abstracts.

- **`transcripts/<role-id>/` parallel to `notes/<role-id>/`.** Three folder conventions now compose. Per-role flat. Multi-role transcripts pick a primary-role folder; secondary participants appear in `participants[]` only. Cross-role concerns deferred (matches v0.3 posture).

- **Working-repo root anchoring** (resolved Q9). Not GitHub-repo root. The clarification was necessary because the proposal text initially said only "parallel to `notes/<role-id>/`" without specifying which root — a genuine ambiguity when the memodef working directory is nested inside a wrapping GitHub repo. ccc-ninja@0.12.0's behavior surfaced the gap; revision closes it.

- **`ended` SHOULD be ABSENT while ongoing** (resolved Q3). Tightening from "SHOULD update once on session close" to "SHOULD be ABSENT while ongoing; populated exactly once on detected close". Empirical motivation: mid-session-snapshot population by ccc-ninja@0.12.0 demonstrated that loose semantics defeat the append-mode property. Stricter SHOULD preserves the design intent.

- **Markdown as canonical body content via body_ref.** ccc-ninja already produces formatted markdown from Claude Code JSONL — speaker labels, code blocks, timestamps. Markdown is the durable form (human-readable, paste-friendly for snippets, exposes literal conversation text); JSONL is the unstable substrate (Anthropic-controlled, format may evolve). Adopters MAY use other formats via `capture_format` field.

- **Capture-tool subject-derivation guidance (CONTRIBUTING-level, not SCHEMA MUST).** Capture tools SHOULD skip harness/system tags (`<ide_opened_file>`, `<system-reminder>`, etc.) when picking the subject anchor; reach for first substantive user turn. This is quality guidance, not normative SCHEMA — tools that emit noisy subjects produce conformant but degraded transcripts.

- **Strategist/maintainer scope-split implementation.** Decision artifact authored by `memodef-strategist`; SCHEMA.md / README.md edits + conformance fixtures authored by `memodef-maintainer`. Same precedent as body_ref v0.2 and memos-to-file v0.3 — a session arc playing both roles with hat-distinction preserved in commit authorship per [CLAUDE.md](../CLAUDE.md).

## Items NOT incorporated

- **Per-conversation folder** (`transcripts/<conversation-id>/`). Rejected — per-role flat composes more naturally with `notes/<role-id>/` precedent; avoids the "who allocates conversation ids" infrastructure question.
- **Sentinel `to: "transcript"` on `memodef:Memo`.** Rejected per Design rationale (structural shape differs from memo).
- **`body` field as short subject-shaped abstract.** Rejected per Director's no-abstract rule and append-mode requirement.
- **JSONL as canonical body content.** Rejected — markdown is durable; JSONL is unstable substrate.
- **`abstract` field as OPTIONAL short summary.** Rejected — adopters who need a one-line description use `subject`. Adding a separate field re-introduces regeneration friction.
- **Separate sibling spec `transcriptdef-spec`.** Rejected per Source-project peer review (Director course-correction to "class of memos in memodef").
- **Detailed schema for each `participants[]` entry** (start_time, end_time, role_at_time, etc.). Out of scope for v0.4; defer until empirical motivation. Current loose shape `{position, session_arc OR identity}` adopters can extend via metadata.
- **`session_arc` SHOULD-guidance against bare UUIDs (refinement #4 candidate).** Surfaced during ratification by the GitHub secret-scanner false-positive observation: session UUIDs match the OpenVSX Personal Access Token format pattern (`[0-9a-f]{8}-[0-9a-f]{4}-...`), triggering false-positive secret leaks on public repository pushes. Strategist recommended SHOULD-guidance for `session_arc` and `metadata.sender_session_arc` recommending implementations avoid bare UUID-shaped substrings (prefer truncated, hashed, or non-UUID descriptors). **Director ruling: superfluous; KISS.** "You can put the session arc in the body if you want." The false-positive is resolvable at the GitHub-alert-closure layer (close as false-positive — they're not real credentials); no spec text needed. Adopters who want session context in memos can use body text rather than the structured field. Logged here so a future strategist re-encountering the same friction has Director's reasoning in the audit trail.
- **Spec ships capture tooling.** memodef defines artifact SHAPE; ccc-ninja and future capture tools are reference implementations, not spec deliverables.

## Workflow validation

- The proposal → revision → decision → implementation flow per CONTRIBUTING.md §3 ran cleanly with empirical-implementation in parallel: design conversation 2026-05-19 (Director course-correction recorded in proposal Origin) → proposal filed 2026-05-20 (commit `eb9f762`) → first-implementation by ccc-ninja@0.12.0 same day → three-refinement memo to ccc-ninja-engineer + proposal revision in place (commit `d50773e`) → ccc-ninja@0.13.0 same-day conformance verified → decision filed (this artifact) → implementation queued.
- The two-work-item engagement split pattern Director used 2026-05-20 ("1) spec updates, 2) ccc-ninja-engineer suggestions") kept spec-side and tool-side concerns distinct in the same session arc. Pattern worth re-using when first-implementation experience surfaces spec + tool concerns simultaneously.
- Receiver-commits convention from `decisions/receiver-commits-convention.md` continues as de-facto operational pattern: ccc-ninja-engineer's repo receives the memodef-strategist feedback memo, their next session commits per convention; oagp-strategist's repo receives the family-level MCP-distribution-idea memo, their next session commits per convention.

## Forward-reference resolution

- **memodef-maintainer roledef** (forthcoming) — MUST inherit the additive-only invariant and the version-stamp-consistent-with-features-used rule clarified by this, body_ref v0.2, and memos-to-file v0.3. v0.4 adds: the new-type-vs-sentinel distinction (POP-discipline lens for "shape vs lifecycle").
- **memodef-strategist roledef** (forthcoming) — MUST inherit the first-implementation-folded-back-pre-ratification pattern (now at second application; promotion to CLAUDE.md threshold when third application surfaces). v0.4 adds: the empirical-motivation-block-in-proposal-revision shape established in this proposal's "First-implementation observations" section.
- **openbraid v0 architecture** — Q7 (openbraid API scoping for transcripts) is openbraid's call. Cross-spec coordination memo to openbraid-engineer / future openbraid-strategist if/when transcript-emission-via-openbraid is in scope.
- **ccc-ninja-engineer** — has the feedback memo (commit pending their receiver-side); has shipped @0.13.0 implementing all three refinements; bonus refinement #4 (forward-slash path separators in `source_jsonl`) unsolicited. Future ccc-ninja work is out-of-scope for memodef-spec ratification but reference-implementation status is documented.
- **`attachments_ref` v0.4+ candidate** — body_ref v0.2's forward-pointer remains. Transcripts with multi-file recording (e.g., simultaneous text + audio + screen capture) would strengthen the empirical case if those patterns surface.
- **family-level MCP at oagp.org/mcp** — Director-floated 2026-05-21, parked. Cross-spec coordination memo filed to oagp-strategist (2026-05-21-1140) so the idea surfaces during future family-level architecture discussion. Not memodef-spec's call to drive.

## Runtime test

**Deferred.** Conformance harness for `memodef:Transcript` artifacts is a Known Work Item (v0.5+ alongside conformance harness for memos). Until automated, conformance is validated by:
- Strategist self-validation of envelopes against SCHEMA MUST/SHOULD fields (this proposal: walked during proposal review)
- Real-world implementation conformance (this proposal: ccc-ninja@0.13.0 same-day conformance verification at `memodef/transcripts/memodef-strategist/2026-05-19-1721--...`)

## Strategist memory work items

- **First-implementation-folded-back-pre-ratification pattern: SECOND APPLICATION** (v0.3 → v0.3.1 via openbraid-engineer 2026-05-10 was the first; v0.4 via ccc-ninja@0.12.0 → 0.13.0 is the second). Promote to CLAUDE.md as a documented strategist-discipline pattern if/when third application surfaces. Logged in [project_memodef.md](C:/Users/edsby/.claude/projects/C--Users-edsby/memory/project_memodef.md).
- **Two-work-item engagement-split pattern** (Director's 2026-05-20 split: "1) spec updates, 2) engineer feedback") — useful when first-implementation surfaces both spec-side and tool-side concerns simultaneously. Logged in [project_memodef.md](C:/Users/edsby/.knowledge); re-use when shape recurs.
- **GitHub-secret-scanner false-positive on session UUIDs** (UUID pattern matches OpenVSX PAT format). Applies family-wide (any catdef-family artifact embedding session identifiers). Per Director KISS ruling, no spec text; adopters who hit the same false-positive close as false-positive at GitHub-alert layer. Surface to openbraid / roledef / orgdef strategists when they next push session-flavored content publicly. Logged in [project_memodef.md](C:/Users/edsby/.claude/projects/C--Users-edsby/memory/project_memodef.md).
- **Three-folder-convention parallel composition.** `memos/{inbox,read,archive}/`, `notes/<role-id>/`, `transcripts/<role-id>/` now share working-repo-root placement and per-role-flat shape (where applicable). Folder convention extends a clean pattern; any future folder-convention proposal should compose with this triad. Logged here for future strategist sessions.

## Notes

- **Director-authorized scope expansion.** Same posture as body_ref v0.2 and memos-to-file v0.3: this decision artifact is filed under memodef-strategist authorship; the implementation (SCHEMA, README, conformance fixtures) is filed under memodef-maintainer authorship in a separate commit. The Director's ratification carries implementation through given the strictly additive nature of the change.
- **Bot-identity provisional pending governance ratification.** memodef-strategist bot identity used for this decision; memodef-maintainer bot identity for implementation. Both remain provisional pending memodef.org governance setup.
- **Library state after this PR.** memodef-spec ships three top-level types (`memodef:Memo`, `memodef:Library`, `memodef:Transcript`) and three folder conventions (`memos/{inbox,read,archive}/`, `notes/<role-id>/`, `transcripts/<role-id>/`). Three-axis framing is now canonical: **memos** = inter-position coordination, **memos-to-file** = intra-position distilled context, **transcripts** = intra-position verbatim recording.
- **Bootstrap-arc strategist context.** Filed by the catdef-family bootstrap session arc playing memodef-strategist informally per `bootstrap-deviation.md`. When the memodef-strategist roledef and seat are formally provisioned, the new strategist may revisit this disposition if real-world adoption surfaces gaps.
- **First spec change with implementation conformance verified BEFORE ratification.** Both v0.2 (body_ref) and v0.3 (memos-to-file) had implementation arrive AFTER ratification; v0.4 inverts the sequence — ccc-ninja@0.13.0 conformant before this decision artifact landed. This is the strongest possible empirical confidence: the proposal text is conformant-against-implementation, not just conformant-by-construction.

## Cross-references

- Proposal artifact (filed-and-revised): [proposals/2026-05-20-transcript-type-and-folder-convention.md](../proposals/2026-05-20-transcript-type-and-folder-convention.md)
- Charter (AI-legibility primacy, POP-discipline, no-vendor-advocacy values): [org/memodef-spec-organization.opencatalog](../org/memodef-spec-organization.opencatalog)
- Architecture-validation decision (folder-as-state, POP-discipline canonization, README-vs-SCHEMA placement precedent): [decisions/proposal-2026-04-29-catdef-strategist-architecture-validation.md](proposal-2026-04-29-catdef-strategist-architecture-validation.md)
- body_ref v0.2 decision (additivity precedent, sibling-file convention reused by Transcript): [decisions/proposal-2026-05-01-body-ref-v0.2.md](proposal-2026-05-01-body-ref-v0.2.md)
- Memos-to-file v0.3 decision (POP-discipline shape-vs-lifecycle precedent, two-axis framing predecessor): [decisions/proposal-2026-05-10-memos-to-file-and-notes-folder.md](proposal-2026-05-10-memos-to-file-and-notes-folder.md)
- v0.3.1 implementer-experience clarifications (first-implementation-folded-back pattern first application): [decisions/v0.3.1-implementer-experience-clarifications.md](v0.3.1-implementer-experience-clarifications.md)
- Receiver-commits decision (workflow convention applied to transcript and feedback memo placement): [decisions/receiver-commits-convention.md](receiver-commits-convention.md)
- Bootstrap deviation context: [decisions/bootstrap-deviation.md](bootstrap-deviation.md)
- Empirical implementation evidence (three captures, two ccc-ninja versions, side-by-side conformance trail):
  - `transcripts/memodef-strategist/2026-05-19-1721--memodef-strategist--hey-claude-i-d-like-to-work-on-caliper.{openthing,body.md}` (ccc-ninja@0.13.0, back-fill capture of a 2026-05-19 caliper session; all three refinements conformant; single-participant intra-position shape)
  - `transcripts/memodef-strategist/2026-05-20-1932--memodef-strategist--ide-opened-file-the-user-opened-the-file-untitled.{openthing,body.md}` (ccc-ninja@0.12.0, pre-refinement capture of THIS proposal session preserved as authentic evidence — harness-tag subject, mid-session `ended` population, originally placed outside git tree by 0.12.0; relocated into working-repo-root for git-tracking conformance)
  - `transcripts/memodef-strategist/2026-05-20-1932--memodef-strategist--hi-could-you-read-projects-memodef-and-take-on-the.{openthing,body.md}` (ccc-ninja@0.14.2, **post-refinement re-capture of the SAME session** as the 0.12.0 evidence above; all three refinements conformant AND first canonical multi-participant transcript with Director added to `participants[]` via `identity` field rather than `session_arc`; produces side-by-side before/after empirical artifact for ratification readers)
- ccc-ninja-engineer feedback memo (cross-repo coordination): `s:/projects/ccc-ninja/memos/inbox/2026-05-20-1955--memodef-strategist--ccc-ninja-engineer--v0-4-transcript-emission-feedback.openthing`
- Family-level MCP idea memo to oagp-strategist (cross-spec coordination): `s:/projects/oagp-org/memos/2026-05-21-1140--memodef-strategist--oagp-strategist--family-level-mcp-distribution-idea.openthing`
- Implementation (queued for memodef-maintainer seat): SCHEMA.md (`memodef:Transcript` top-level type definition, MUST/SHOULD fields, internal-consistency rule for body field absence, version bump to 0.4.0); README.md (Transcripts folder section, three-axis framing table, capture-tool integration expectations); conformance/{valid_memos,invalid_memos}/ (canonical transcript pair, multi-role transcript, ongoing-no-ended transcript, transcript without body_ref invalid case, transcript with body field invalid case, transcript with memo fields invalid case)
