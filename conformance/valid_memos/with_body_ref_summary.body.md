== Background ==

This is the sibling body file for `with_body_ref_summary.openthing`. The fixture demonstrates the canonical v0.2 body_ref pattern: a long-form memo body lives here as plain markdown, while the memo envelope's `body` field holds a 1–3-sentence triage summary.

This file is what the memo's `body_ref` field points to. Validators with filesystem access SHOULD verify this file exists at the resolved path.

== Priorities ==

In a real handoff, this section would enumerate Q2 priorities. For fixture purposes, the content here is illustrative:

1. **Analytics platform migration** — finalize vendor decision by 2026-06-01.
2. **Infrastructure capacity planning** — Q3 baseline projections due 2026-06-15.
3. **Vendor consolidation review** — three contracts up for renewal in Q3.

== Open items ==

Four open items from Q1 carrying into Q2:

- Review of last quarter's incident retros — assigned to Bob's team
- Updated runbook for the analytics ETL — owner TBD
- Compliance sign-off on the new vendor agreement — pending legal review
- Onboarding plan for the two Q3 hires — coordinate with HR

== Decisions needed ==

By 2026-06-01:
- Analytics vendor selection (current shortlist of three vendors)
- Infrastructure capacity envelope (whether to expand or hold steady)
- Vendor consolidation: which contracts to renew vs. consolidate

By 2026-06-15:
- Q3 OKRs for the three workstreams above
- Resourcing plan for the Q3 hires

== Done ==

Reply via memo at `<bob-working-directory>/memos/inbox/`, with `in_reply_to` set to this memo's filename. Cover decisions on the four items above plus any open questions.

— Alice
