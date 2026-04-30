# Decision — proposal-2026-04-29-catdef-strategist-architecture-validation

**Disposition:** Accept all three items, with placement adjustments to fit a Claude-to-Claude deployment context. Folder-as-state RECOMMENDED in README.md (not SCHEMA.md). Channel notification shape included in v0.1 README.md as informational (revised from the proposal's "defer entirely" suggestion). POP-discipline canonized in README.md "Design philosophy" + a CLAUDE.md "Operating posture" pointer. No SCHEMA.md changes; no version bump.
**Origin:** Strategist-pending [proposals/2026-04-29-catdef-strategist-architecture-validation.md](../proposals/2026-04-29-catdef-strategist-architecture-validation.md), surfacing peer memo [memos/2026-04-29-1700--catdef-strategist--memodef-strategist--architecture-validation-and-pop-discipline.openthing](../memos/2026-04-29-1700--catdef-strategist--memodef-strategist--architecture-validation-and-pop-discipline.openthing).
**Decided:** 2026-04-26 by memodef-strategist (currently played by the catdef-family bootstrap session arc per [decisions/bootstrap-deviation.md](bootstrap-deviation.md)).
**Validator:** N/A — proposal items are workflow conventions and design framing, not schema changes; no MUST/SHOULD walk applicable. Inbound peer memo schema-validated as Pass-with-notes (see Notes).

## Source-project peer review

The peer-strategist memo (`from: catdef-strategist`, `to: memodef-strategist`, sent 2026-04-29) is the validating peer review for v0.1 architecture and the source of the three proposed additions. The memo arrived independently — drafted from a separate session arc without access to memodef working state — and reaches the v0.1 architectural conclusions by deduction from first principles. That fresh-eyes-converge signal is itself a recordable bootstrap-validation artifact.

## Detailed disposition

### Item 1 — Folder-as-state convention (maildir-style)

**Accepted as RECOMMENDED in [README.md → Inbox lifecycle](../README.md#inbox-lifecycle-recommended), not SCHEMA.md.**

SCHEMA.md is reserved for memo SHAPE (the load-bearing artifact). Folder layout is workflow. Conflating them sets a precedent inviting future workflow rules into the schema. README.md placement is the right home and reversible if real-world adoption diverges.

The Claude-first benefit is load-bearing: a receiving Claude session answers "what do I need to deal with?" with one filesystem call (`ls memos/inbox/`). The cost (one commit per state-change via `git mv`) is small relative to that inspection-cost savings, and git captures the lifecycle for free.

### Item 2 — Channel notification minimum shape

**Accepted; included in [README.md → Notification, in practice](../README.md#notification-in-practice) as informational, with Claude Code Channels named as the canonical instantiation.**

This is a revision from the proposal's "defer entirely to memodef-notify" suggestion. The revision is grounded in the human-steward directive that memodef's primary purpose is Claude-to-Claude communication with a human-readable git-preserved chain. With Claude as the deployment context, naming the Claude Code primitive is documenting where memos actually live, not vendor advocacy. The runtime-agnostic principle ("notification carries the path; content lives in the file") is stated alongside, preserving the equal-citizen rule for the spec while serving the actual adopter base.

Notification-layer protocol details (semantics of file-watcher events, MCP server messages, RSS feed format, SMTP envelopes) remain deferred to the memodef-notify Known Work Item ([CONTRIBUTING.md](../CONTRIBUTING.md)).

### Item 3 — POP-discipline framing

**Canonized.** Two homes:
- [README.md → Design philosophy: POP-like](../README.md#design-philosophy-pop-like) — visible to adopters and contributors; sets expectations before they author enrichment proposals we'll then have to decline
- [CLAUDE.md → Operating posture](../CLAUDE.md#operating-posture) — adds a pointer as the lens for strategist-level enrichment-pressure decisions

The earned-experience context (the human steward's IMAP / X.400 / MAPI MTA implementation background) is preserved here in rationale rather than in the canonized README.md text — keeps the framing impersonal in adopter-visible docs while giving future strategist sessions the load-bearing context.

### Process question (Q5 from proposal)

**Surfacing as `proposals/` is case-by-case, not the default.** Incoming peer-strategist memos that contain proposals → surface as proposal artifacts (this case). Pure FYI memos → live in `memos/`, no artifact. Defaulting all peer memos into `proposals/` would inflate the queue with non-actionable items.

## Rationale

The dispositions hew to a Claude-first lens (per human-steward directive): memodef's primary use case is Claude-to-Claude communication with a human-readable git-preserved chain. That:

- **Shifts Item 2.** Naming Claude Code Channels in the README is documenting the deployment substrate, not preferring a vendor — and it gives Claude sessions a concrete entry point.
- **Reinforces Item 1.** Maildir-style inbox is a Claude-friendly affordance: one `ls` call answers "what do I need to deal with?" — the alternative (read every memo to determine state) is materially worse for token-bounded sessions.
- **Reinforces Item 3.** A POP-floor-lean substrate is what lets Claude sessions in fresh repos with minimal scaffolding produce and consume conformant memos. Enrichment that adds infrastructure burden harms Claude flows specifically.

## Notable design choices

- **README.md vs SCHEMA.md placement.** Workflow conventions (folder-as-state) and design framing (POP-discipline) belong in README.md. SCHEMA.md is reserved for memo SHAPE. Worth preserving as precedent in future dispositions.
- **Channels named, but framed as instantiation.** README.md states the runtime-agnostic principle first; Claude Code Channels is presented as one canonical instantiation among possible alternatives. Equal-citizen for the spec, useful for the actual adopter base.
- **No version bump.** All changes are README.md and CLAUDE.md additions — informational/editorial, not normative changes to SCHEMA.md. memodef remains v0.1.0.
- **Two-commit split on the implementing PR.** Decision artifact authored by `memodef-strategist`; README.md/CLAUDE.md edits authored by `memodef-maintainer`. Preserves the strategist/maintainer scope distinction in git history, even when the same session arc plays both roles during bootstrap.

## Items not incorporated

- **`== Section ==` body convention** for the inbound memo — not addressed by this disposition; flagged as Pass-with-notes during validation. Not material to the proposal items.
- **Earned-experience caveat as canonized README.md text** — preserved here in rationale instead, keeping README.md framing impersonal.
- **Full memodef-notify scope** — Item 2 includes a Channels-instantiation note; full notification-layer protocol semantics remain deferred.
- **Strict mark-as-read enforcement** — RECOMMENDED, not REQUIRED. Adopters with high-volume positions or non-git tooling MAY use a flat `memos/`.

## Workflow validation

- The `proposals/` workflow served as a clean pickup mechanism for strategist-pending peer-memo input, even though the items weren't in the standard "schema change" form. Open–Discuss–Iterate–Decide–Implement–VersionBump from CONTRIBUTING.md §3 fit.
- The maintainer/strategist hat-switching during bootstrap (per bootstrap-deviation.md) functioned cleanly: maintainer-scope filing of the proposal artifact distinct from strategist-scope decision filing, with bot-identity differentiation in commit authorship.

## Forward-reference resolution

- **memodef-strategist roledef** (forthcoming derivation of `senior-open-standards-strategist`) — when authored, should incorporate the POP-discipline lens as a stated canon, with cross-reference to this decision and the catdef-strategist's 2026-04-29 memo. The roledef is the durable home for strategist-discipline framing once it exists.
- **memodef-strategist memory file** (provisional, not yet provisioned) — when set up, should carry forward the POP-discipline lens and the Item 1 / Item 2 dispositions as default context.

## Notes

- **Validation outcome on inbound memo:** Pass-with-notes. Conforms to all MUST fields and most SHOULD fields. SHOULD note: body uses `# Heading` Markdown rather than the [SCHEMA.md `== Section ==` convention](../SCHEMA.md#L156). Content nit (not a schema concern): "Scott has built IMAP, X.400, and IMAP MTAs in his career" — IMAP appears twice; likely meant MAPI given the surrounding protocol list. Not material to the disposition.
- **Earned-experience context** (preserved per rationale): the human steward has built IMAP, X.400, and MAPI Mail Transfer Agents over their career. The POP framing is anchored in that implementation experience, not naïveté. Future strategist sessions reading this decision should treat the POP-discipline lens as a strategic discipline grounded in implementation history, not a fallback or concession.
- **Bootstrap-arc strategist context.** This decision is filed by the catdef-family bootstrap session arc playing memodef-strategist informally (per bootstrap-deviation.md). When the memodef-strategist roledef and seat are formally provisioned, the new strategist may revisit any item if real-world adoption surfaces gaps.

## Cross-references

- Inbound peer memo: [memos/2026-04-29-1700--catdef-strategist--memodef-strategist--architecture-validation-and-pop-discipline.openthing](../memos/2026-04-29-1700--catdef-strategist--memodef-strategist--architecture-validation-and-pop-discipline.openthing)
- Strategist-pending tracking artifact: [proposals/2026-04-29-catdef-strategist-architecture-validation.md](../proposals/2026-04-29-catdef-strategist-architecture-validation.md)
- Bootstrap deviation context: [decisions/bootstrap-deviation.md](bootstrap-deviation.md)
- README.md sections: [Design philosophy](../README.md#design-philosophy-pop-like), [Notification, in practice](../README.md#notification-in-practice), [Inbox lifecycle](../README.md#inbox-lifecycle-recommended)
- CLAUDE.md section: [Operating posture](../CLAUDE.md#operating-posture)
- Notification-layer Known Work Item: [CONTRIBUTING.md](../CONTRIBUTING.md)
