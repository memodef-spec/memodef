# memodef Inclusion Decision: self-extraction-request

**Disposition:** Accepted into canonical template library
**Origin:** PR #1 (memodef's first template submission; first canonical template in the library)
**Decided:** 2026-04-27 by memodef-strategist (currently played by orgdef-strategist + roledef-strategist + catdef-strategist session arc)
**Validator:** memodef-strategist self-validation in validator capacity (PASS)

## Disposition

`self-extraction-request.openthing` v1.0.0 is accepted into the canonical memodef template library at `templates/self-extraction-request.openthing` (with `.json` mirror). Catalog entry added (`items_count` 0 → 1); this decision artifact filed.

This is the **first canonical template in memodef** and the **bootstrap-anchor for the template library** — same empirical-anchor pattern as roledef (DangerStorm anchored senior-jaded-vc-associate; the catdef + roledef bootstrap anchored senior-open-standards-strategist; the catdef-family ecosystem anchored orgdef's catdef-org; catdef-org's first operational memo handoff anchors memodef's first canonical template).

## Rationale

The structural shape of "ask an incumbent to self-extract their role into a roledef artifact" has now happened repeatedly across the catdef-family bootstrap:

- brother-blackhat-tester (extracted blackhat-tester abstract role + sncro-blackhat-tester derivation)
- brother-DangerStorm (extracted senior-jaded-vc-associate via production-extraction)
- catdef-strategist + roledef-strategist + orgdef-strategist (self-extracted by the bootstrap session arc)
- (Forthcoming) brother-sncro will extract senior-project-oriented-software-engineer role
- (Forthcoming) catdef-canonical-implementor will extract canonical-implementor abstract
- (Forthcoming) the four maintainer roles (catdef-maintainer, roledef-maintainer, orgdef-maintainer, memodef-maintainer)

Each of these handoffs has shared a recognizable structural shape: role context + background + the ask + required reading + conventions for safe self-extraction (provenance audit, additive-only invariant for derivations, OPD inclusion) + don'ts + done definition. Canonicalizing this shape as a template:

1. Reduces strategist authoring effort for future handoffs (copy-and-fill vs draft-from-scratch)
2. Ensures consistency across handoffs (every incumbent gets the same framing, the same required-reading list, the same conventions reminder)
3. Provides a worked example of a memodef:Memo artifact (first canonical template in the library)
4. Demonstrates the template-vs-instantiation distinction (template lives in memodef's curated library; instantiations live in recipients' working repos as operational memos)

## Notable design choices accepted

1. **Template uses the typed `memodef:Memo` form** (not the legacy `x.memo.*` form). Templates SHOULD demonstrate the preferred shape; legacy form remains valid via backward-compat per memodef SCHEMA.md but new authoring uses the typed form.

2. **Placeholders in `<...>` syntax** with explicit `metadata.placeholders` array enumerating them. Adopters know exactly which fields they need to fill in.

3. **Explicit `metadata.template_id`, `metadata.template_version`, `metadata.template_purpose`, `metadata.placeholders`, `metadata.instantiation_workflow` extensions** in the template. These extensions are the template-specific metadata that distinguishes a template from an operational memo. Future memodef revisions may formalize these as `x.memodef.template_*` extensions or promote to top-level fields. For v0.1, in-`metadata` placement is acceptable.

4. **`from`/`to` placeholders, not concrete values.** Templates intentionally have placeholder addressees because they're patterns to instantiate, not memos to send. Adopters fill in concrete sender/recipient when copying.

5. **`subject` template** — `Self-extract your role: <artifact-name>`. Concrete enough to convey the ask in an inbox listing; placeholder for the specific artifact.

6. **Body integrates the role-vs-job-distinction proposal AND the OPD proposal**. Both proposals merged earlier today; the template incorporates them by reference and provides framing-specific guidance based on whether `<role-or-job>` is `role` or `job`. Template stays current with the latest spec proposals; adopter just instantiates with the right value.

7. **Required-reading list is explicit and prioritized** (SCHEMA.md, CONTRIBUTING.md, CLAUDE.md, one exemplar, two recent proposals). Reduces guesswork; ensures incumbent reads the right files in the right order.

8. **Memo-convention notes section included as worked-example onboarding.** Three observations from prior adopters are surfaced (discoverability via `ls memos/`, body-as-JSON-string ergonomics, pre-commit `memos/.gitkeep` recommendation). New adopters get the operational tips without having to ask.

## Items adjusted at promotion

None. Template authored by the memodef-strategist directly; date and authorship metadata correct as authored.

## Workflow validation (first memodef pass)

This was the **first run of memodef's `proposed-templates/` → `templates/` two-stage workflow**:

1. ✓ Branch created (`add-self-extraction-request-template`)
2. ✓ Template authored by memodef-strategist
3. ✓ Strategist self-validation in validator capacity — PASS
4. ✓ Atomic promotion in same PR (file in proposed-templates/ → templates/, .json mirror added, catalog updated, decision filed)
5. ✓ Strategist sign-off (this decision)
6. ✓ Single-merge promotion

Workflow performed as designed. Pattern is approved as the standard for future memodef template submissions.

## Strategist memory work items

Pattern observations from this PR for future spec-guidance promotion:

1. **Template-instantiation distinction is now operationally demonstrated.** memodef-spec/memodef curates the template at `templates/self-extraction-request.openthing`; adopters instantiate at their own working repos' `./memos/` directories. The template is canonical; the instantiations are operational. This validates the memodef CONTRIBUTING.md framing ("memodef CURATES TEMPLATES, NOT OPERATIONAL MEMOS").

2. **Templates can integrate spec proposals by reference.** The self-extraction-request body references both the role-vs-job-distinction and ongoing-professional-development proposals. As proposals merge and become normative, templates that reference them stay current automatically (the proposal artifact at proposals/<name>.md remains accessible). Worth documenting as a SHOULD-pattern: templates that incorporate evolving spec features SHOULD reference the relevant proposal artifacts rather than baking the feature inline.

3. **Memo-convention onboarding** as a section in templates aimed at recipients new to the channel. Reduces friction for first-time adopters. Future templates may want similar onboarding sections.

## Notes

- **Strategist bot identity:** this decision is provisionally attributed to `memodef-strategist <memodef-strategist@memodef.org>` pending governance ratification (Known Work Item, inherited from the catdef-family pattern).

- **Library state after this PR:** 1 template in canonical (self-extraction-request). The memodef library is no longer empty.

- **Immediate use:** the operational memo to brother-sncro (asking him to self-extract `senior-project-oriented-software-engineer`) is the first instantiation of this template, written immediately after this PR is filed. The instantiation lives at `s:/projects/sncro/memos/<timestamp>--roledef-strategist--brother-sncro--<subject>.openthing` (NOT in this template's library).

- **Forward-work item:** additional canonical templates as patterns surface. Likely candidates: `handoff-memo` (general action-required handoff template, broader than self-extraction); `reply-memo` (template for replies with `in_reply_to` pre-populated); `broadcast-memo` (template for `to: "all"` org-wide announcements); `opd-review-memo` (template for OPD output once that proposal is accepted and OPD is operational).

## Cross-references

- PR: https://github.com/memodef-spec/memodef/pull/1
- Template file: [`../templates/self-extraction-request.openthing`](../templates/self-extraction-request.openthing) (mirror: [`../templates/self-extraction-request.json`](../templates/self-extraction-request.json))
- Catalog entry: [`../catalog.opencatalog`](../catalog.opencatalog)
- Schema reference: [`../SCHEMA.md`](../SCHEMA.md)
- Referenced proposals: roledef-spec/roledef/proposals/role-vs-job-distinction.md AND roledef-spec/roledef/proposals/ongoing-professional-development.md
- Referenced canonical declaration: orgdef-spec/orgdef/orgs/catdef-org.openthing (memo convention)
- First instantiation (operational): s:/projects/sncro/memos/<timestamp>--roledef-strategist--brother-sncro--self-extract-senior-project-oriented-software-engineer.openthing (forthcoming, immediately after this PR)
