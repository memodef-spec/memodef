# Contributing to memodef

memodef is an open standard plus a community-curated reference library of memo TEMPLATES. This document describes how to contribute — both new templates to the library and proposed changes to the spec.

**Important:** memodef does NOT curate operational memos. Memos are operational artifacts that live in their recipient orgs' per-position working repos (per the org-declared memo location convention). The memodef library curates memo TEMPLATES — exemplar artifacts demonstrating canonical patterns for adopters to copy from.

## Governance

memodef is stewarded independently of any single AI vendor or implementation, following the catdef-family pattern:

- **Vendors do not own the spec.** All AI runtimes are equal citizens.
- **The library is curated; the spec is open.** Anyone can author memodef artifacts at any URL using the same format.
- **Bounded AI authority.** AI sessions draft, validate, decide, sign off; human maintainers merge.

## Two kinds of contribution

1. **Template submissions** — adding a new memo template to `templates/`
2. **Spec proposals** — proposing changes to the memodef schema, the contribution process, or governance rules

## 1. Template submissions

### What templates are

A template is a `memodef:Memo` artifact (or `memodef:Library` index) that demonstrates a canonical pattern. Examples: a handoff template (action-required + structured body), a reply template (with `in_reply_to` populated and threading conventions), a broadcast template (`to: "all"`), an archival template, a security-classified template.

Templates are intended to be COPIED. An adopter authoring an operational memo SHOULD start from the closest matching template, fill in the placeholders, save to the recipient's `./memos/` inbox.

### The two-stage workflow

Template submissions follow the catdef-family two-stage pattern:

- `proposed-templates/` — submissions in flight; publicly visible but not yet promoted to canonical
- `templates/` — the canonical template library; entries here have passed validation and review

**Submission steps:**

1. **Fork** the repo
2. **Author your template** following the schema in [SCHEMA.md](SCHEMA.md)
3. **Add the file** to `proposed-templates/<your-template-id>.openthing`
4. **Open a PR** against `main`
5. **Validation runs** — produces a validation report posted on the PR
6. **Maintainer review** — checks scope, quality, library fit, attribution
7. **Strategist sign-off** — required for borderline cases
8. **Promotion at merge** — atomic move from `proposed-templates/` to `templates/`, catalog update, decision artifact filed

### Validation expectations

A template submission MUST pass:

- **Schema validation** — all MUST fields present per [SCHEMA.md](SCHEMA.md), correct catdef wrapping, valid versioning, no reserved namespace misuse
- **Demonstrable pattern** — the template must demonstrate a CONCRETE pattern useful to adopters (handoff, reply, broadcast, etc.). Generic "a generic memo" templates are rejected for redundancy with `SCHEMA.md` examples.

A template submission SHOULD pass:

- **Placeholder discipline** — fields that adopters fill in are clearly marked (e.g., `<sender position id>`, `[insert subject here]`)
- **Pattern annotation** — top-of-body comment explaining what pattern the template demonstrates and when an adopter would use it

### Quality bar

The library is curated. Even valid templates may be rejected for:

- **Redundancy** — the library already has a template covering this pattern
- **Scope** — too narrow (only useful for one specific org) or too broad (just rephrases SCHEMA.md examples)
- **Quality** — vague placeholders, unclear pattern, contradictory guidance

If your template is rejected for the canonical library, you can publish it at any URL using the same format.

## 2. Operational memos (NOT a contribution path)

Operational memos are not contributions to memodef. They live in their recipient orgs' per-position working repos per the org-declared memo location convention (e.g., the catdef-org convention is `<recipient_position.x.position.working_directory>/memos/`).

If your org wants to declare a memo-location convention, do that in your `orgdef:Organization` artifact's `x.org.memo_location` extension. memodef defines the memo TYPE; orgdef defines the per-org memo LOCATION.

## 3. Spec proposals

Changes to the memodef schema, the contribution process, or governance rules use a separate workflow via the `proposals/` directory.

Same shape as catdef / roledef / orgdef proposals. Each proposal: Summary, Motivation, Proposed Change, Backward Compatibility, Conformance Tests, Alternatives Considered, Open Questions.

### Proposal workflow

1. **Open** — file the proposal as a PR adding `proposals/<name>.md`
2. **Discuss** — open period for community/maintainer/strategist input
3. **Iterate** — refine based on feedback
4. **Decide** — the memodef-strategist files a decision in `decisions/proposal-<name>.md`
5. **Implement** (if accepted) — the memodef-maintainer drafts the actual schema change as a follow-on PR
6. **Version bump** — accepted spec changes bump the memodef schema version per [SCHEMA.md](SCHEMA.md#versioning)

## Test suite (conformance)

Conformance fixtures live in `conformance/`:

- `valid_memos/` — exemplar memos demonstrating proper schema usage
- `invalid_memos/` — counterexamples demonstrating common errors validators must catch
- `runtime_evidence/` — per-runtime test outputs (deferred to v0.2+ when memodef-aware tooling lands)

## Strategist bot identity

The memodef-strategist role (forthcoming as a derivation of [`senior-open-standards-strategist`](https://roledef.org/roledefs/senior-open-standards-strategist.openthing)) operates under the bot identity `memodef-strategist <memodef-strategist@memodef.org>` for commits and decision-artifact authorship. Provisional pending governance ratification (Known Work Item, inherited from the catdef-family pattern).

## Cross-spec coordination

When a memodef-level decision affects another catdef-family spec:

- **catdef substrate concerns** → coordinate with catdef-strategist
- **roledef-reference concerns** (e.g., the `from`/`to` fields reference position ids that come from orgdef positions which reference roledefs) → coordinate with roledef-strategist
- **orgdef-convention concerns** (e.g., memo-location convention shape that orgdef artifacts declare) → coordinate with orgdef-strategist

The bounded-authority discipline applies across spec boundaries.

## Known work items inherited from catdef-family

- **Strategist bot identity ratification** — provisional pending governance
- **Validator-as-CI-step automation** — until then, strategist self-validates
- **Auto-generation of `catalog.opencatalog`** — currently hand-maintained
- **Conformance test harness** — runtime-aware testing infrastructure deferred to v0.2+
- **Notification-layer spec** — memodef defines memos as files; the protocols for notifying recipients of new memos (file-watcher / MCP-as-notification / RSS / SMTP) are deferred to v0.2+ or to a separate memodef-notify spec
