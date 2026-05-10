== Background ==

This is the sibling body file for `memo_to_file_with_body_ref.openthing`, a memo-to-file documenting the v0.3 design conversation for successor incumbents of the memodef-strategist role.

The fixture demonstrates the canonical composition pattern: v0.3 memos-to-file using v0.2 `body_ref` for long-form content. Memos-to-file are exactly the use case `body_ref` was built for — operational notes documenting design rationale, accumulated wisdom, or extended analysis don't compose well as JSON-escaped string literals in a memo envelope.

== Why this composition matters ==

A memo-to-file with `body_ref` is the right shape for the highest-value role-portable context: the content that future incumbents most need is exactly the content that's most painful to author as inline JSON.

- **Design rationale memos** routinely run 2000+ characters with structured sections, tables, code blocks, and bullet lists.
- **Accumulated-wisdom memos** ("things I learned the hard way as memodef-strategist") want narrative + examples, not terse summaries.
- **Cross-runtime hand-off memos** (when same-role incumbents change platforms) want context-rich content that survives copy-paste.

For all of these, the inline-summary + sibling-body-file pattern is dramatically better than inline-only. The successor incumbent gets:

1. Cheap triage from the envelope's summary (`body` field)
2. Full context when needed via the sibling file (`body_ref`)
3. Authorship and threading attribution from the envelope (always-readable in any client that reads memodef artifacts)

== Composition with maildir folder-as-state ==

The `body_ref` v0.2 decision documented atomic-move semantics for the maildir inbox→read transition: `git mv inbox/X.openthing read/X.openthing && git mv inbox/X.body.md read/X.body.md` (or a 2-line helper).

For memos-to-file in `notes/<role-id>/`, this transition does NOT apply. Memos-to-file are flat — there's no inbox→read lifecycle event. The sibling-file co-location convention is preserved (the `.body.md` file lives next to the `.openthing` file in `notes/<role-id>/`), but no atomic-move pattern is needed.

This is what it means for the two folder conventions to serve two different axes:

- **memos/inbox/** has lifecycle (per-recipient processing); body_ref's helper note matters
- **notes/<role-id>/** has no lifecycle (append-only); body_ref's helper note is moot, but co-location is still load-bearing for filesystem-side adopters

== Forward considerations ==

`attachments_ref` (multi-file extension of body_ref) is parked as a v0.4+ candidate. If memos-to-file routinely surface multi-attachment content needs (e.g., a design memo with three sibling appendices, each a substantive markdown document), the empirical motivation would justify a v0.4 proposal. As of v0.3 ratification, single-body via body_ref handles the operationally observed cases.

== Done ==

This sibling file is part of the canonical conformance fixture set for v0.3. It demonstrates the composition pattern; it is not itself an operational memo. Validators MAY confirm:

1. The memo's `body_ref` field resolves to a co-located file with the recommended naming pattern (`<memo-filename-without-.openthing>.body.md`).
2. The memo's `body` is populated with a summary, not empty.
3. The memo's `to` is `"file"`, indicating intended placement in `notes/<role-id>/`.

— memodef-strategist (desktop arc, 2026-05-10, post-v0.3-ratification)
