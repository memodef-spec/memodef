# Decision — receiver-commits convention canonized (RECOMMENDED)

**Disposition:** Canonize the receiver-commits convention in [README.md → Receive convention (RECOMMENDED)](../README.md). When a memo arrives in a recipient's working repo (placed by sender via filesystem write, network share, or other transport), the **receiver** commits the memo to git as receipt-of-record. Commit author = receiver's seat bot identity. Convention is RECOMMENDED, not REQUIRED — adopters whose tooling expects sender-commits MAY use that pattern.
**Origin:** Open question Q2 from [memos/read/2026-04-30-1230--memodef-strategist--memodef-strategist--workflow-observations-on-desktop-arc.openthing](../memos/read/2026-04-30-1230--memodef-strategist--memodef-strategist--workflow-observations-on-desktop-arc.openthing). No preceding proposal artifact filed; this decision stands alone, following the [`bootstrap-deviation.md`](bootstrap-deviation.md) precedent for strategist calls grounded in accumulated empirical evidence rather than a formal proposal.
**Decided:** 2026-05-01 by memodef-strategist (currently played by the catdef-family bootstrap session arc per [decisions/bootstrap-deviation.md](bootstrap-deviation.md)), Director-ratified same day.
**Validator:** N/A — workflow convention, not a schema change. No MUST/SHOULD walk against the inbound memo applicable; the open question's lifecycle was tracked in strategist-side memory rather than as a formal proposal artifact.

## Source-project peer review

The convention emerged organically across the catdef-family bootstrap arc rather than via a single peer proposal. Four data points across two repos (memodef-spec and orgdef-spec) all converge on receiver-commits:

| # | Date | Memo | Pattern |
|---|---|---|---|
| 1 | 2026-04-29 | catdef-strategist → memodef-strategist (architecture validation) | Sender placed memo on receiver's filesystem unstaged; receiver committed |
| 2 | 2026-04-30 | orgdef-strategist → memodef-strategist (derivation review) | Same — receiver committed (laptop arc, directly to memos/read/ as per-action shortcut) |
| 3 | 2026-04-30 | memodef-strategist → memodef-strategist (workflow observations) | Same — receiver committed before pushing |
| 4 | 2026-05-01 | orgdef-strategist → memodef-strategist (body_ref case-2 trigger) | Same — receiver committed (commit `dd23017`) |

The data is consistent and the pattern is converging. Q2 of the laptop-arc workflow-observations memo flagged this as "Worth canonizing or revisiting deliberately"; canonization is the call.

## Detailed disposition

**Receiver-commits is RECOMMENDED.** Specifically:

1. The sender writes the memo file to the recipient's working repo using whatever transport is appropriate (direct filesystem write under the catdef-org authorization-implies-write-access discipline; or network share; or paste-on-receive when sessions overlap).
2. The receiver, on session start or memo arrival, runs `git status` (or equivalent) to discover untracked memo files in their inbox.
3. The receiver `git add`s the new memo file(s) and commits with a "Receive <subject>" or equivalent imperative-subject message. **Commit author = receiver's seat bot identity** (e.g., `memodef-strategist <memodef-strategist@memodef.org>`), preserving the strategist/maintainer scope distinction in git history.
4. The receiver pushes the commit on a cadence that fits their session arc (immediately, or batched with substantive response work).

The sender is **not expected** to commit to the recipient's repo. The sender's responsibility ends when the memo file is delivered to the recipient's working tree. Subsequent lifecycle (commit, mark-as-read, archive) is the recipient's seat-scope.

## Rationale

- **Workflow simplicity.** Senders writing across repos already need filesystem write access to the recipient's working tree (per the authorization-implies-write-access discipline). Adding commit-and-push to the sender's responsibilities would couple the sender to the recipient's repo lifecycle (need to know which branch is current, when to push, how to handle PR-based workflows on that repo, etc.). Receiver-commits keeps each seat's lifecycle local to its own repo.
- **Authorship semantics.** The receiver's commit captures the receive event in git history with the receiver's seat as commit author. This preserves "who acted on this memo" as a queryable git-log property — `git log --author=memodef-strategist memos/` shows exactly the receive-record of the memodef-strategist seat, regardless of who sent the memos.
- **AI-legibility primacy.** A future Claude session pointed at the recipient's repo can answer "what's in my inbox?" with a single `git log` or `git status` plus filesystem inspection. Sender-commits would mix sender-authored history with receiver-authored history, complicating the audit trail's seat-by-seat partitioning.
- **No infrastructure cost.** Receiver-commits requires nothing the receiver doesn't already do (commit lifecycle in their own repo). Sender-commits would require senders to maintain push permissions and push-cadence discipline for every recipient repo they communicate with.

## Notable design choices

- **RECOMMENDED, not REQUIRED.** Adopters whose tooling expects sender-commits (e.g., a CI bot that auto-commits incoming memos on the sender side, or a sync-service-mediated transport) MAY use that pattern. The canonical convention is receiver-commits because it's the cheapest discipline; deviation is permitted with reason.
- **Receiver as commit author, not just committer.** Git distinguishes author (the original change-maker) from committer (the entity producing the commit object). Per CLAUDE.md bot-identity discipline, AI-maintainer / AI-strategist commits use the seat's bot identity as **author**. The receiver's seat is the author of receipt-of-record commits.
- **Two-step receive-and-process is permitted but not required.** A receiver MAY commit the memo to `memos/inbox/` first (receipt-of-record commit), then `git mv` to `memos/read/` after processing (mark-as-read commit) — two commits, clearer audit trail. OR a receiver MAY commit directly to `memos/read/` when receive-and-process happen in one action (the laptop arc's per-action shortcut, defensible). Both patterns are valid; the audit trail still captures the lifecycle correctly.
- **Sender's filename-of-record convention applies.** The sender chooses the memo's filename per the [SCHEMA.md file naming convention](../SCHEMA.md#file-naming-convention-recommended). The receiver MUST NOT rename the file on receipt (that would break `in_reply_to` references).

## Items not incorporated

- **Sender-commits-and-pushes alternative as default.** Rejected — couples sender to recipient's repo lifecycle, violates the principle that each seat's repo is its own scope.
- **Mandatory receipt-of-record commit before mark-as-read.** SHOULD-not-MUST per the per-action-shortcut precedent (laptop arc 2026-04-30). A single receive-and-mark-as-read commit is permitted when receive and process happen atomically.
- **Schema change for receive-state metadata.** Rejected — receive state is workflow, expressed through git history, not memo schema. Adding `metadata.received_at` or similar would conflate transport state with memo content.

## Workflow validation

- The convention has been operationally executed across 4+ data points without surfacing failure modes. The pattern works in both single-arc (laptop or desktop) and dual-arc (laptop + desktop) configurations of the bootstrap session.
- The receiver-commits + mark-as-read + push sequence composes naturally with the maildir folder-as-state convention canonized in `proposal-2026-04-29-catdef-strategist-architecture-validation.md`.
- The bot-identity discipline (commit author = seat) is preserved cleanly: receive-record commits show in git log under the receiver's seat identity, regardless of which session arc holds the seat.

## Forward-reference resolution

- **memodef-maintainer roledef** (forthcoming) — when authored, MUST inherit the receiver-commits convention as a workflow rule.
- **memodef-notify spec** (Known Work Item) — when authored, MUST treat memo arrival as a delivery-to-filesystem event with subsequent receiver-commits as the audit trail; notification protocols MUST NOT couple to commit timing on the sender side.
- **Cross-org `from`/`to` qualification syntax** (Known Work Item) — when formalized, the receiver-commits convention applies regardless of qualification: a memo from `catdef-org/roledef-strategist` to `memodef-org/memodef-strategist` lives in memodef's repo and is committed by the memodef-strategist seat.

## Notes

- **Q1 (where memos land — flat vs inbox/) and Q3 (typed vs legacy form) remain open.** Q1 is converging on sender-places-at-inbox-when-recipient-adopts-convention but has only 2 clear data points and the discoverability mechanism (orgdef extension `x.org.memo_inbox`) is an orgdef-strategist proposal candidate, not memodef's. Q3 has mixed data; cross-family migration is not memodef's call. Both stay in carry-forward state until 3+ data points converge per question.
- **No corresponding proposal artifact filed.** Standalone decisions are the right home when the underlying call accumulated through accumulated lived experience rather than a formal proposal cycle. The bootstrap-deviation.md precedent is the model.
- **Bootstrap-arc strategist context.** This decision is filed by the catdef-family bootstrap session arc playing memodef-strategist informally per `bootstrap-deviation.md`. When the memodef-strategist roledef and seat are formally provisioned, the new strategist may revisit this disposition if real-world adoption surfaces gaps.

## Cross-references

- Open-question source: [memos/read/2026-04-30-1230--memodef-strategist--memodef-strategist--workflow-observations-on-desktop-arc.openthing](../memos/read/2026-04-30-1230--memodef-strategist--memodef-strategist--workflow-observations-on-desktop-arc.openthing)
- Implementation: README.md → Receive convention (RECOMMENDED) section
- Related conventions: [README.md → Inbox lifecycle (RECOMMENDED)](../README.md#inbox-lifecycle-recommended), [README.md → Where memos live](../README.md#where-memos-live)
- Bootstrap deviation context: [decisions/bootstrap-deviation.md](bootstrap-deviation.md)
- CLAUDE.md bot-identity discipline: [CLAUDE.md → Reserved conventions](../CLAUDE.md)
- Architecture-validation decision (folder-as-state, POP-discipline): [decisions/proposal-2026-04-29-catdef-strategist-architecture-validation.md](proposal-2026-04-29-catdef-strategist-architecture-validation.md)
