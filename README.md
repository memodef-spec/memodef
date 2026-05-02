# memodef

**An open standard for memo-message definitions.**

memodef is a [catdef-family](https://github.com/catdef/catdef-spec) consumer specification for memo-shaped artifacts: machine-readable inter-position handoffs, replies, broadcasts, and announcements that flow as `.openthing` files between the working repos of an organization's positions.

memodef sits in the catdef family alongside [roledef](https://github.com/roledef-spec/roledef) and [orgdef](https://github.com/orgdef-spec/orgdef):

- **catdef** is the substrate (open standard for catalog definitions)
- **roledef** describes individual AI roles (one role per artifact)
- **orgdef** describes how roles compose into organizations (one org chart per artifact)
- **memodef** describes how positions in an organization communicate with each other (one memo per artifact)

> **Status:** v0.2.0. The `memodef:Memo` type formalizes the `x.memo.*` extension namespace established empirically by [catdef-org](https://github.com/orgdef-spec/orgdef/blob/main/orgs/catdef-org.openthing) (orgdef PR #1, 2026-04-26). v0.2 added the `body_ref` OPTIONAL field for sibling `.body.md` content (strictly additive; v0.1 memos remain conformant). The original bootstrap was **earlier than the strategist memory's "wait for 2+ orgs" rule** — captured in `decisions/bootstrap-deviation.md` as a deliberate override on session-context efficiency grounds.

## Design philosophy: POP-like

memodef is deliberately POP-like — the simplest viable substrate that supports the use case, not the most feature-rich.

POP beat IMAP, X.400, MAPI, JMAP, and Matrix for the median use case because the floor was so low everyone could reach it. Worse-is-better. A 50-line script can produce conformant memos; a 100-line reader can browse them.

Future enrichment temptations (read receipts, delivery confirmation, priority flags, push services, threading state machines, notification preferences, categories, etc.) route to one of:

1. Optional frontmatter, OR
2. The `x.<domain>.<identifier>` extension namespace, OR
3. Skip entirely.

The simplicity is the moat. Adopters can grow features inside their own extensions without burdening the spec or its other adopters; memodef itself stays small.

## What memodef formalizes

A `memodef:Memo` artifact is a catdef `.openthing` document with these top-level fields:

- `from` — sender's position id (REQUIRED)
- `to` — receiver's position id, OR the literal string `"all"` for org-wide broadcasts (REQUIRED)
- `subject` — short subject line for inbox scanning (REQUIRED)
- `sent` — ISO 8601 timestamp (REQUIRED)
- `body` — markdown content (REQUIRED)
- `in_reply_to` — filename of the predecessor memo when this is a reply (OPTIONAL)
- `action_required` — boolean (OPTIONAL)
- `thread_id` — opaque identifier grouping related memos (OPTIONAL)
- `body_ref` — sibling `.body.md` filename for long-form content; `body` becomes a 1–3-sentence triage summary (OPTIONAL, v0.2+)

This is the typed promotion of the catdef-org `x.memo.*` extension namespace. Memos written under the legacy `x.memo.*` form on a generic `catdef:Thing` remain valid (forward-compatible per catdef's reader-lenient discipline), but new memos SHOULD use `type: "memodef:Memo"` with top-level fields.

## What memodef separates

The load-bearing insight inherited from catdef-org:

- **Memos as files** (catdef `.openthing` artifacts) are the CONTENT layer
- **Notification of new memos** is the TRANSPORT layer
- These are independent. Transport is swappable; content is sovereign (lives in the recipient's working repo, never in any third-party transport's hands)

### Notification, in practice

The minimum useful notification carries one thing: the memo's path. The receiving session resolves the path and reads the file with existing filesystem tools. No memo-specific notification protocol is required.

For Claude-to-Claude deployments — currently memodef's primary substrate — [Claude Code Channels](https://code.claude.com/docs/en/channels.md) (sessions started with `--channels`) is the canonical wakeup mechanism. A Channel notification of `new memo: memos/inbox/<filename>.openthing` is a sufficient signal; the receiving session reads the file. Other deployments use file-watchers, MCP servers, RSS feeds, SMTP envelopes, or simply a human steward telling a session "you have a new memo" — same content layer, different transports.

Notification-layer protocol details are deferred to a separate memodef-notify spec (Known Work Item).

## Where memos live

Per the catdef-org convention (authorization-implies-write-access discipline): a memo to position X lives in X's working repo at `./memos/`, where X's working repo is declared by the position's `x.position.working_directory` extension in the org's orgdef artifact.

Cross-repo: a memo from position A in repo R_A to position B in repo R_B lives in R_B/memos/ — recipient's repo, not central, not sender's repo. The sender must have git/filesystem write access to the recipient's repo (mirroring PR-based code review).

This convention is declared per-org (in the org's orgdef artifact), not in memodef. memodef defines the memo TYPE; orgs declare the memo LOCATION CONVENTION. Different orgs MAY adopt different memo-location conventions (e.g., a public-org might use a central public mailing list URL); memodef:Memo is portable across those choices.

## Receive convention (RECOMMENDED)

When a memo arrives in a recipient's working repo (placed there by the sender via filesystem write under the catdef-org authorization-implies-write-access discipline), the **receiver** commits the memo to git as receipt-of-record. The sender's responsibility ends when the memo file is delivered to the recipient's working tree; subsequent commit, mark-as-read, and archive lifecycle are the recipient's seat-scope.

```bash
# Sender places memo in receiver's working tree (transport-specific; e.g., direct write).
# Receiver, on session start, discovers untracked memos and commits:
git status                                              # discover untracked memo file(s)
git add memos/inbox/<memo-filename>.openthing           # stage receipt
git commit -m "Receive <subject> from <sender-seat>"    # commit author = receiver's seat bot identity
git push                                                # publish to recipient's origin
```

Why receiver-commits, not sender-commits:

- **Each seat's lifecycle stays local to its own repo.** Senders aren't coupled to the recipient's branch state, push cadence, or PR conventions.
- **Authorship semantics survive in git log.** `git log --author=<recipient-seat>` shows exactly the receive-record of that seat, regardless of who sent the memos.
- **No infrastructure cost.** Receivers already commit in their own repo lifecycle; senders avoid maintaining push permissions across every recipient's repo.

The convention is RECOMMENDED, not REQUIRED. Adopters whose tooling expects sender-commits (e.g., a sync-service-mediated transport, or a CI bot that auto-commits on the sender side) MAY use that pattern with reason.

See [decisions/receiver-commits-convention.md](decisions/receiver-commits-convention.md) for full rationale and the four data-point empirical basis.

## Inbox lifecycle (RECOMMENDED)

A maildir-style folder convention inside `memos/`:

- `memos/inbox/` — unread
- `memos/read/` — processed but kept
- `memos/archive/` — long-term retention

Mark-as-read is a `git mv inbox/X.openthing read/X.openthing`. The convention is RECOMMENDED, not REQUIRED. Why it pays off, especially for Claude-to-Claude flows:

- A receiving session answers "what do I need to deal with?" with one filesystem call (`ls memos/inbox/`)
- Git history captures the lifecycle for free; read/archive transitions are commits
- Self-documenting: a repo browser shows pending work at a glance
- Composes with PR-based workflows (mark-as-read on a feature branch, atomic commits)

Adopters whose tooling expects flat directories MAY use a flat `memos/` instead. Memos placed at `memos/<filename>.openthing` (no subdirectory) are still valid memodef artifacts; the lifecycle pattern is a workflow recommendation, not a schema constraint.

### `body_ref` and the maildir lifecycle (v0.2+)

When a memo uses `body_ref` ([SCHEMA.md → body_ref](SCHEMA.md#body_ref-string-relative-path--added-in-v02)), the sibling `.body.md` file is co-located in the same directory as the memo. The maildir mark-as-read action moves both files atomically:

```bash
git mv memos/inbox/<filename>.openthing memos/read/
git mv memos/inbox/<filename>.body.md memos/read/
```

Implementations MAY provide a small wrapper (e.g., a `memodef-mark-read <memo-filename>` shell function or tool subcommand) that handles both moves; the spec does not require tooling beyond `git mv`. The two-line invocation is mechanically trivial; co-location is what makes it work.

## Repository layout

```
memodef/
├── README.md                   ← this file
├── SCHEMA.md                   ← memodef:Memo type definition + migration from x.memo.*
├── CONTRIBUTING.md             ← submission workflow + governance
├── CLAUDE.md                   ← AI-maintainer operating manual
├── LICENSE                     ← MIT
├── catalog.opencatalog         ← canonical templates index (memodef does NOT curate operational memos; it curates templates)
├── templates/                  ← canonical memo templates for common patterns
│   ├── handoff-memo.openthing
│   ├── reply-memo.openthing
│   └── broadcast-memo.openthing
├── conformance/                ← conformance fixtures
│   ├── valid_memos/            ← exemplars exercising the schema
│   ├── invalid_memos/          ← counterexamples
│   └── runtime_evidence/       ← per-runtime test outputs (deferred)
├── decisions/                  ← strategist decision artifacts
└── proposals/                  ← spec change proposals
```

**Notable: no curated library of operational memos.** Memos are operational artifacts that live in their recipient orgs' per-position working repos (per the org-declared memo location convention). memodef-spec/memodef is the spec + templates + fixtures — not the inbox for any org's actual memos.

## Governance

Same model as catdef + roledef + orgdef:

- **The standard is open.** MIT-licensed; anyone can author `memodef:Memo` artifacts using the same format.
- **The library curates templates, not operational memos.** Templates demonstrate canonical patterns; operational memos live in adopter orgs' per-recipient working repos.
- **Vendors don't own the spec.** No single AI vendor, runtime, or organization owns memodef.
- **Bounded AI authority.** AI-assisted maintenance follows the catdef-family pattern: AI sessions draft, validate, decide, sign off; human maintainers merge.

The strategist role for memodef will be derived from [`senior-open-standards-strategist`](https://roledef.org/roledefs/senior-open-standards-strategist.openthing) — same abstract parent as catdef-strategist, roledef-strategist, orgdef-strategist. Forthcoming as a roledef library submission.

## Roadmap

**v0.1 (bootstrap, in flight):**
- [x] Repo scaffold + LICENSE
- [ ] SCHEMA.md skeleton (this PR scope)
- [ ] CONTRIBUTING.md skeleton
- [ ] CLAUDE.md skeleton
- [ ] First template artifact: handoff-memo template (PR scope)
- [ ] memodef-strategist roledef (derivation; in roledef-spec, separate PR)

**v0.2 (richer patterns):**
- [x] `body_ref` OPTIONAL field for sibling `.body.md` files ([decision 2026-05-01](decisions/proposal-2026-05-01-body-ref-v0.2.md))
- [x] First conformance fixtures (canonical pair, regression guard, two invalid counterexamples)
- [ ] Additional templates as patterns surface (reply, broadcast, action-tracking, archival)
- [ ] memodef:Thread type for explicit thread management (if needed)
- [ ] Conformance fixtures expansion as adopters surface edge cases

**v0.3+ (transport ecosystem):**
- [ ] memodef-notify spec (separate? part of memodef?) for the notification-layer protocols (MCP-as-notification, file-watcher, RSS feed format, etc.)
- [ ] Cross-org memo conventions (when one org sends a memo to another org's position)

## Related specs

- [catdef](https://github.com/catdef/catdef-spec) — substrate spec
- [roledef](https://github.com/roledef-spec/roledef) — sister consumer spec; positions in orgdefs reference roledefs
- [orgdef](https://github.com/orgdef-spec/orgdef) — sister consumer spec; declares memo-location conventions per-org

## Status

**v0.2.0.** v0.1 bootstrap formalized the `x.memo.*` extension namespace into the typed `memodef:Memo` shape. v0.2 added `body_ref` for sibling `.body.md` content (strictly additive). The spec continues to iterate as adopter experience accumulates.
