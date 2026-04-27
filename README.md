# memodef

**An open standard for memo-message definitions.**

memodef is a [catdef-family](https://github.com/catdef/catdef-spec) consumer specification for memo-shaped artifacts: machine-readable inter-position handoffs, replies, broadcasts, and announcements that flow as `.openthing` files between the working repos of an organization's positions.

memodef sits in the catdef family alongside [roledef](https://github.com/roledef-spec/roledef) and [orgdef](https://github.com/orgdef-spec/orgdef):

- **catdef** is the substrate (open standard for catalog definitions)
- **roledef** describes individual AI roles (one role per artifact)
- **orgdef** describes how roles compose into organizations (one org chart per artifact)
- **memodef** describes how positions in an organization communicate with each other (one memo per artifact)

> **Status:** v0.1 bootstrap. The `memodef:Memo` type formalizes the `x.memo.*` extension namespace established empirically by [catdef-org](https://github.com/orgdef-spec/orgdef/blob/main/orgs/catdef-org.openthing) (orgdef PR #1, 2026-04-26). The bootstrap is **earlier than the strategist memory's "wait for 2+ orgs" rule** — captured in `decisions/bootstrap-deviation.md` as a deliberate override on session-context efficiency grounds.

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

This is the typed promotion of the catdef-org `x.memo.*` extension namespace. Memos written under the legacy `x.memo.*` form on a generic `catdef:Thing` remain valid (forward-compatible per catdef's reader-lenient discipline), but new memos SHOULD use `type: "memodef:Memo"` with top-level fields.

## What memodef separates

The load-bearing insight inherited from catdef-org:

- **Memos as files** (catdef `.openthing` artifacts) are the CONTENT layer
- **Notification of new memos** (today: human-steward; tomorrow: file-watcher / MCP-as-notification / RSS / SMTP) is the TRANSPORT layer
- These are independent. Transport is swappable; content is sovereign (lives in the recipient's working repo, never in any third-party transport's hands)

## Where memos live

Per the catdef-org convention (authorization-implies-write-access discipline): a memo to position X lives in X's working repo at `./memos/`, where X's working repo is declared by the position's `x.position.working_directory` extension in the org's orgdef artifact.

Cross-repo: a memo from position A in repo R_A to position B in repo R_B lives in R_B/memos/ — recipient's repo, not central, not sender's repo. The sender must have git/filesystem write access to the recipient's repo (mirroring PR-based code review).

This convention is declared per-org (in the org's orgdef artifact), not in memodef. memodef defines the memo TYPE; orgs declare the memo LOCATION CONVENTION. Different orgs MAY adopt different memo-location conventions (e.g., a public-org might use a central public mailing list URL); memodef:Memo is portable across those choices.

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

**v0.1 bootstrap.** The spec is being authored alongside the first template artifact. Both will iterate as we discover what the schema actually needs to support.
