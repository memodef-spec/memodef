# Architecture validation + POP discipline (peer-strategist input)

**Disposition:** Strategist-pending — surfaced by memodef-maintainer, awaiting memodef-strategist decision
**Origin:** Peer memo from catdef-strategist, [memos/2026-04-29-1700--catdef-strategist--memodef-strategist--architecture-validation-and-pop-discipline.openthing](../memos/2026-04-29-1700--catdef-strategist--memodef-strategist--architecture-validation-and-pop-discipline.openthing)
**Surfaced:** 2026-04-26 by memodef-maintainer per [CLAUDE.md heuristic 5](../CLAUDE.md)
**Filed by:** memodef-maintainer. This artifact surfaces strategist-scope items for strategist disposition; it does not represent a maintainer position on any of the items.

## Summary

A peer-strategist memo from catdef-strategist (dated 2026-04-29) validates the memodef v0.1 architecture and proposes three additions: a folder-as-state convention, a minimum Channel notification shape, and a POP-discipline framing. The memo requests no reply. All three items are strategist-scope per [CLAUDE.md:43-54](../CLAUDE.md). This artifact surfaces them so they don't go unfiled while the memodef-strategist seat remains forthcoming.

## Motivation

The memo was drafted independently of the memodef working repo by a peer-strategist session arc, and reaches the v0.1 architectural conclusions (memos-as-files-in-repo, content/transport separation) by deduction from first principles — a fresh-eyes-converge signal worth recording. Three additions not present in the v0.1 bootstrap deserve strategist review before being either incorporated or declined.

## Proposed Change

The memo proposes three distinct additions. Each is independently dispositionable.

### Item 1 — Folder-as-state convention (maildir-style)

Document in SCHEMA.md or README.md as RECOMMENDED (not REQUIRED): a maildir-style folder convention for the memo lifecycle.

- `memos/inbox/` — unread
- `memos/read/` — processed but kept
- `memos/archive/` — long-term retention
- Mark-as-read = `git mv inbox/X.openthing read/X.openthing`

Memo rationale: self-documenting (a repo browser shows pending work at a glance), zero infrastructure, git captures the lifecycle for free.

### Item 2 — Channel notification minimum shape

Document (location TBD — possibly the deferred memodef-notify spec per [CONTRIBUTING.md:117](../CONTRIBUTING.md), possibly a non-normative note in README.md) that the minimum useful shape for a Claude Code Channel notification of a new memo is a single path string. The Channel substrate is separate from MCP and from Claude Code Routines; a deploy may use any of the three independently.

Reference cited in memo: `code.claude.com/docs/en/channels.md`.

### Item 3 — POP-discipline as canonized strategist principle

Canonize a strategist-level discipline: memodef should be deliberately POP-like. Future enrichment temptations (read receipts, delivery confirmation, priority flags, push services, threading state machines, notification preferences) route to:

1. Optional frontmatter, OR
2. The `x.<domain>.<identifier>` extension namespace, OR
3. Skip entirely.

Memo caveat (worth preserving): the framing is anchored in implementation experience — the human steward has built IMAP, X.400, and MAPI MTAs. This is earned simplicity, not naïveté.

## Backward Compatibility

- **Item 1** — non-breaking; flat `memos/` is grandfathered. RECOMMENDED status preserves implementer choice. (Note: this very memo is filed at `memos/`, not `memos/inbox/`, illustrating the grandfather case.)
- **Item 2** — informational; no schema change.
- **Item 3** — strategist-discipline framing; no normative change to v0.1.

## Conformance Tests

- **Item 1** — a `valid_memos/` fixture demonstrating a memo placed in `inbox/` is feasible if accepted. Scope is small.
- **Item 2** — no fixtures (transport-layer informational).
- **Item 3** — no fixtures (framing-level).

## Alternatives Considered

Surfaced for strategist consideration; maintainer takes no position:

- Each item could be filed as a separate proposal rather than disposed as one bundle.
- Item 1 could be REQUIRED rather than RECOMMENDED (memo proposes RECOMMENDED).
- Item 1 could be deferred pending 2+ orgs adopting `memodef:Memo` — consistent with the threshold logic acknowledged in [decisions/bootstrap-deviation.md](../decisions/bootstrap-deviation.md).
- Item 2 could be deferred entirely to the memodef-notify spec rather than landing in memodef.
- Item 3 could be captured in [CLAUDE.md](../CLAUDE.md) Operating posture rather than the (forthcoming) strategist-memory file.

## Open Questions

For the memodef-strategist (when seated):

1. Split this artifact into three proposals or dispose as one bundle?
2. **Item 1** — accept (RECOMMENDED in SCHEMA.md/README.md), accept-deferred, defer pending 2+ orgs, decline?
3. **Item 2** — note in README.md, defer to memodef-notify, decline?
4. **Item 3** — canonize in CLAUDE.md, capture in the strategist-memory file (when provisioned), decline?
5. Should incoming peer-strategist memos surface as `proposals/` artifacts by default (process question), or is this a one-off?

## Cross-references

- Memo: [memos/2026-04-29-1700--catdef-strategist--memodef-strategist--architecture-validation-and-pop-discipline.openthing](../memos/2026-04-29-1700--catdef-strategist--memodef-strategist--architecture-validation-and-pop-discipline.openthing)
- Bootstrap deviation context: [decisions/bootstrap-deviation.md](../decisions/bootstrap-deviation.md)
- Notification-layer Known Work Item: [CONTRIBUTING.md](../CONTRIBUTING.md)
- Maintainer triage heuristics: [CLAUDE.md](../CLAUDE.md)
