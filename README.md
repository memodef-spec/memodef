# memodef

**An open standard for memo-message definitions.**

memodef is a [catdef-family](https://github.com/catdef/catdef-spec) consumer specification for memo-shaped artifacts: machine-readable inter-position handoffs, replies, broadcasts, and announcements that flow as `.openthing` files between the working repos of an organization's positions.

memodef sits in the catdef family alongside [roledef](https://github.com/roledef-spec/roledef) and [orgdef](https://github.com/orgdef-spec/orgdef):

- **catdef** is the substrate (open standard for catalog definitions)
- **roledef** describes individual AI roles (one role per artifact)
- **orgdef** describes how roles compose into organizations (one org chart per artifact)
- **memodef** describes how positions in an organization communicate with each other (one memo per artifact)

> **Status:** v0.3.1. The `memodef:Memo` type formalizes the `x.memo.*` extension namespace established empirically by [catdef-org](https://github.com/orgdef-spec/orgdef/blob/main/orgs/catdef-org.openthing) (orgdef PR #1, 2026-04-26). v0.2 added the `body_ref` OPTIONAL field for sibling `.body.md` content. v0.3 added the `to: "file"` sentinel and `notes/<role-id>/` folder convention for **intra-position context portability**. v0.3.1 added implementer-experience clarifications driven by openbraid's first-implementation report (SHOULD-violation surfacing, metadata-passthrough, hosted-store folder encoding) — clarifications only, no behavior change. Strictly additive throughout; v0.1 / v0.2 / v0.3.0 memos remain conformant. The original bootstrap was **earlier than the strategist memory's "wait for 2+ orgs" rule** — captured in `decisions/bootstrap-deviation.md` as a deliberate override on session-context efficiency grounds.

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
- `to` — receiver's position id, OR `"all"` for org-wide broadcasts, OR `"file"` for memos-to-file filed for accumulated role context (REQUIRED, `"file"` added in v0.3)
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

## Notes folder — memos-to-file (RECOMMENDED, v0.3+)

A second folder convention parallel to `memos/`, for **role-portable accumulated context**:

- `notes/<role-id>/` — memos filed for a role's record (not directed at any position; flat, no inbox/read/archive lifecycle)
- One folder per role: `notes/memodef-strategist/`, `notes/catdef-strategist/`, etc.
- Lifecycle is "append-only via commit history" (or, in hosted-store transports like [openbraid](https://openbraid.app), append-only log)
- No mark-as-read — every successor incumbent reads everything they want to inherit

Memos in this folder use the `to: "file"` sentinel ([SCHEMA.md → `to`](SCHEMA.md#to-string)). The role context comes from folder placement, not a separate field; `metadata.sender_session_arc` becomes load-bearing for distinguishing successive incumbents who share `from = <role-id>`.

The three folder conventions (memos + notes + transcripts; see [Transcripts folder](#transcripts-folder--verbatim-recordings-recommended-v04) below) serve three axes:

| Axis | Type | Folder | Lifecycle | Audience |
|---|---|---|---|---|
| Inter-position coordination | `memodef:Memo` | `memos/inbox/` → `read/` → `archive/` | Maildir (per-recipient processing) | AI sessions (handoffs, action requests, replies, broadcasts) |
| Intra-position context portability | `memodef:Memo` with `to: "file"` | `notes/<role-id>/` | Flat (append-only) | Successor AI incumbents of the same role |
| Verbatim recording (v0.4+) | `memodef:Transcript` | `transcripts/<role-id>/` | Flat, append-only | Director (snippets, recovery) + AI sessions (lost-window recovery, audit-trail pinning) |

A single repo MAY use any combination, all three, or none. Neither is REQUIRED; the conventions are workflow recommendations, not schema constraints. Memos placed at flat `memos/` (no subdirectory) and role-folders elided are still valid memodef artifacts.

**Hosted-store implementations** (e.g., [openbraid](https://openbraid.app)) encode the folder convention through query-time discrimination (a `kind` column or equivalent) rather than filesystem paths. The wire shape and behavioral semantics are unchanged: `notes/<role-id>/` maps to `WHERE kind='note' AND role_id=<role-id>` (or equivalent) in non-filesystem substrates. Adopters of hosted-store implementations interact via the implementation's API surface (e.g., MCP tool calls); the memo SHAPE is identical to the git-substrate case.

The motivating use case (operationally tested across Claude Code + Claude Desktop + Claude-on-iPhone + Perplexity): a human interacts with what feels like one durable role (e.g., "I'm talking to catdef-strategist"), but behind that role any number of agents play across runtimes and over multiple days. Memos-to-file in `notes/<role-id>/` preserve accumulated context across role-incumbents so the human's mental model tells the truth.

See [decisions/proposal-2026-05-10-memos-to-file-and-notes-folder.md](decisions/proposal-2026-05-10-memos-to-file-and-notes-folder.md) for design rationale.

## Transcripts folder — verbatim recordings (RECOMMENDED, v0.4+)

A third folder convention parallel to `memos/` and `notes/<role-id>/`, for **verbatim conversation recordings**:

- `transcripts/<role-id>/` — `memodef:Transcript` envelopes + their sibling `.body.md` files (paired, both co-located)
- The `transcripts/` folder lives at the **working-repo root** — the same directory level as `CLAUDE.md`, `SCHEMA.md`, `README.md`, `memos/`, `decisions/`, `org/`. In a repo where the memodef working directory is nested below the GitHub-repo root, `transcripts/` lives inside the working directory (matching `memos/` and `notes/`), not at the GitHub-repo root
- One folder per role: `transcripts/memodef-strategist/`, `transcripts/openbraid-engineer/`, etc.
- Multi-role conversations live in the **primary role's** folder; secondary participants appear in `participants[]` only
- Lifecycle: append-only; the sibling `.body.md` grows as the conversation progresses (capture tool regenerates or appends), the envelope's `ended` field is set exactly once on detected session close and is otherwise absent (see [SCHEMA.md → memodef:Transcript SHOULD rules](SCHEMA.md#recommended-fields-should--memodeftranscript))
- No mark-as-read — transcripts are not directed to a recipient who processes; the audience is "future readers" (Director, AI successors)

Memos and transcripts serve different audiences over different content shapes. Memos are **distillations** (the AI's interpretation, structured for triage); transcripts are **verbatim recordings** (the literal exchange, structured for citation and recovery). The body_ref pattern from v0.2 is reused: envelope is metadata-only, content lives in the sibling `.body.md`. Markdown is the canonical body content; capture tools (e.g., [ccc-ninja](https://github.com/scottconfusedgorilla/ccc-ninja)) produce formatted markdown from runtime session logs.

**Three real use cases** drove the v0.4 ratification:
1. Director snippets for books, presentations, citations — the transcript is the source-of-truth.
2. Lost-window recovery — when a Claude Code window dies, a fresh session reads `transcripts/<role-id>/<arc>.body.md` to reconstruct the working state.
3. Audit-trail pinning — decisions citing "the rationale was discussed at length" pin to transcript filenames; verbatim source is queryable.

**Capture-tool integration expectations.** Capture tools SHOULD emit a `memodef:Transcript` envelope + sibling `.body.md` as an atomic pair. The envelope can be auto-populated from session metadata the tool already has access to (participants, started, transport, capture_tool, capture_format); subject and related_memos may require human or AI annotation. Tools SHOULD derive `subject` from substantive conversational content (typically the first user turn after harness/system bootstrap), not from harness metadata or transport-injected tags. Tools that emit only the markdown body without an envelope produce conformant content but require manual envelope authoring by the receiver before the transcript is fully addressable.

**Hosted-store implementations** (e.g., openbraid) encode the folder convention through query-time discrimination (a `kind` column or equivalent) rather than filesystem paths — same posture as `notes/<role-id>/` per v0.3.1. The wire shape and behavioral semantics are unchanged.

See [decisions/proposal-2026-05-20-transcript-type-and-folder-convention.md](decisions/proposal-2026-05-20-transcript-type-and-folder-convention.md) for design rationale (including the new-type-vs-sentinel distinction under POP-discipline) and empirical motivation (ccc-ninja@0.12.0 → 0.13.0 → 0.14.2 same-day conformance trail).

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
├── memos/                      ← inter-position coordination (per-recipient maildir lifecycle)
│   ├── inbox/                  ← unread (per-recipient)
│   ├── read/                   ← processed but kept
│   └── archive/                ← long-term retention
├── notes/                      ← intra-position context portability (v0.3+, flat per role)
│   └── <role-id>/              ← memos-to-file for one role; append-only
├── transcripts/                ← intra-position verbatim recordings (v0.4+, flat per role)
│   └── <role-id>/              ← memodef:Transcript envelopes + sibling .body.md pairs
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

**v0.3 (intra-position context portability):**
- [x] `to: "file"` sentinel + `notes/<role-id>/` folder convention ([decision 2026-05-10](decisions/proposal-2026-05-10-memos-to-file-and-notes-folder.md))
- [x] Conformance fixtures for memos-to-file (basic, threading, body_ref composition, action-required violation)
- [x] Two-axis framing canonized in README (memos = inter-position coordination; memos-to-file = intra-position context portability)

**v0.4 (verbatim recording):**
- [x] `memodef:Transcript` top-level type for verbatim conversation recordings ([decision 2026-05-21](decisions/proposal-2026-05-20-transcript-type-and-folder-convention.md))
- [x] `transcripts/<role-id>/` folder convention at working-repo root, parallel to `notes/<role-id>/`
- [x] Three-axis framing canonized in README (memos = inter-position coordination; memos-to-file = intra-position distilled context; transcripts = intra-position verbatim recording)
- [x] First spec change with implementation conformance verified BEFORE ratification (ccc-ninja@0.13.0, @0.14.2)
- [x] Conformance fixtures for transcripts (canonical, multi-role, ongoing-no-ended, three invalid counterexamples)

**v0.5+ (transport ecosystem and beyond):**
- [ ] memodef-notify spec (separate? part of memodef?) for the notification-layer protocols (MCP-as-notification, file-watcher, RSS feed format, etc.)
- [ ] Cross-org memo conventions (when one org sends a memo to another org's position)
- [ ] `attachments_ref` for multi-file body content (parked from body_ref v0.2 Q5; awaiting empirical motivation)
- [ ] Conformance harness (validator-as-CI, runtime-aware testing)

## Related specs

- [catdef](https://github.com/catdef/catdef-spec) — substrate spec
- [roledef](https://github.com/roledef-spec/roledef) — sister consumer spec; positions in orgdefs reference roledefs
- [orgdef](https://github.com/orgdef-spec/orgdef) — sister consumer spec; declares memo-location conventions per-org

## Status

**v0.3.1.** v0.1 bootstrap formalized the `x.memo.*` extension namespace into the typed `memodef:Memo` shape. v0.2 added `body_ref` for sibling `.body.md` content. v0.3 added the `to: "file"` sentinel and `notes/<role-id>/` folder convention for intra-position context portability — operationally tested across Claude Code + Claude Desktop + Claude-on-iPhone + Perplexity with [openbraid](https://openbraid.app) as the hosted-store transport. v0.3.1 added implementer-experience clarifications from openbraid's first-implementation report (no behavior changes). All changes strictly additive. The spec continues to iterate as adopter experience accumulates.
