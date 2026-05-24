== Context for successor sessions ==

You are the next memodef-strategist incumbent. This memo-to-file captures the v0.4 ratification arc 2026-05-20/21. Read this before substantive v0.5+ work so the institutional context is in your head before reading any single decision artifact.

== What shipped ==

**v0.4.0 — `memodef:Transcript` top-level type + `transcripts/<role-id>/` folder convention.**

Strictly additive. v0.1 / v0.2 / v0.3 / v0.3.1 memos remain conformant. Three top-level types now in the spec (`memodef:Memo`, `memodef:Library`, `memodef:Transcript`) and three folder conventions at working-repo root (`memos/{inbox,read,archive}/`, `notes/<role-id>/`, `transcripts/<role-id>/`).

Three-axis framing is canonical:

| Axis | Type | Folder |
|---|---|---|
| Inter-position coordination | memodef:Memo | memos/inbox/ → read/ → archive/ |
| Intra-position distilled context | memodef:Memo with `to: "file"` | notes/<role-id>/ |
| Intra-position verbatim recording | memodef:Transcript | transcripts/<role-id>/ |

Director's three load-bearing use cases for the verbatim recording axis: (1) snippets for books/presentations, (2) lost-window recovery ("I HATE THAT"), (3) audit-trail pinning of distillations to verbatim source.

== Commits, in order ==

| Commit | Hat | What |
|---|---|---|
| `eb9f762` | strategist | Filed v0.4 proposal (initial) |
| `d50773e` | strategist | Revised v0.4 proposal + relocated orphaned 0.12.0 transcript pair into working-repo-root (was outside git tree) |
| `289fd07` | strategist | Filed decision artifact + ccc-ninja@0.14.2 multi-participant transcript |
| `70203d0` | maintainer | SCHEMA.md + README.md + 6 conformance fixtures |

== Pattern: first-implementation-folded-back-pre-ratification — SECOND APPLICATION ==

v0.3 → v0.3.1 via openbraid-engineer 2026-05-10 was the first application. v0.4 via ccc-ninja@0.12.0 → 0.13.0 → 0.14.2 same-day is the second. The shape:

1. Strategist files proposal
2. Real implementation ships against proposal text ahead of ratification
3. First implementation surfaces concrete gaps proposal authors missed
4. Strategist folds refinements back into proposal text BEFORE the decision artifact lands
5. Strategist files decision; implementer ships conformant build same day
6. Ratification gets to weigh in on text that has been touched by reality

**If a third application surfaces, promote to CLAUDE.md as a documented strategist-discipline pattern.** Track it in the strategist memory file ([C:/Users/edsby/.claude/projects/C--Users-edsby/memory/project_memodef.md]) until then.

== The ccc-ninja conformance trail ==

The capture-loop closed on itself: ccc-ninja@0.12.0 captured the very session that drafted the v0.4 proposal, surfacing three concrete refinements:

| Refinement | 0.12.0 behavior | 0.13.0 / 0.14.2 behavior |
|---|---|---|
| Subject derivation | Grabbed `<ide_opened_file>` harness tag | First substantive user turn |
| `ended` field | Populated mid-session, advancing on re-save | Absent until detected close |
| Folder anchoring | GitHub-repo root (OUTSIDE git tree in this project's layout) | Working-repo root, inside git tree |

All three refinements folded back into the proposal pre-ratification, then ccc-ninja-engineer shipped @0.13.0 same day implementing all three, then @0.14.2 added multi-participant shape (Director added to `participants[]` via `identity` field). Filed a typed feedback memo at `s:/projects/ccc-ninja/memos/inbox/2026-05-20-1955--memodef-strategist--ccc-ninja-engineer--v0-4-transcript-emission-feedback.openthing` — three refinements + six load-bearing-precedent confirmations.

Empirical evidence preserved in repo:
- `transcripts/memodef-strategist/2026-05-19-1721--...--hey-claude-i-d-like-to-work-on-caliper.{openthing,body.md}` — ccc-ninja@0.13.0, back-fill caliper session, all three refinements
- `transcripts/memodef-strategist/2026-05-20-1932--...--ide-opened-file-the-user-opened-the-file-untitled.{openthing,body.md}` — ccc-ninja@0.12.0, **pre-refinement capture preserved as authentic evidence** (subject deliberately not corrected; original noisy slug retained as before/after artifact)
- `transcripts/memodef-strategist/2026-05-20-1932--...--hi-could-you-read-projects-memodef-and-take-on-the.{openthing,body.md}` — ccc-ninja@0.14.2, post-refinement re-capture of the SAME session as the 0.12.0 evidence; side-by-side empirical artifact + first canonical multi-participant transcript

== Items NOT incorporated (with reasoning preserved) ==

- **`session_arc` SHOULD-guidance against bare UUIDs (refinement #4 candidate).** GitHub-secret-scanner false-positives on Claude Code session UUIDs (UUID format matches OpenVSX Personal Access Token pattern). Strategist recommended SHOULD-guidance for `session_arc` and `metadata.sender_session_arc` to avoid bare UUID-shaped substrings (truncated, hashed, or non-UUID descriptors). **Director ruling: superfluous; KISS.** Quote: "You can put the session arc in the body if you want." False-positive is resolvable at GitHub-alert-closure layer; no spec text needed. Recorded in decision artifact "Items NOT incorporated" so a future strategist re-encountering the same friction has Director's reasoning in the audit trail.

  Successor-relevant: if the same false-positive recurs and a different strategist seat wants to reopen, the Director ruling stands unless empirical evidence accumulates that body-text workaround is insufficient (e.g., multiple adopters report friction). Track silently; do not re-propose without empirical signal.

- **Refinement #4 in the engineer-feedback memo (forward-slash path separators in `source_jsonl`).** Bonus unsolicited refinement ccc-ninja-engineer added when shipping 0.13.0. Cosmetic but platform-portable; not normative.

- **Federated per-spec MCPs at oagp.org** (Director-floated idea, parked). Original variant rejected by Director — "means adding 4 MCP tools every time" defeats the friction-reduction motivation. Settled shape: one MCP at `oagp.org/mcp` with per-spec tool namespaces, single registration, per-spec strategist owns slice. Memo filed at `s:/projects/oagp-org/memos/2026-05-21-1140--memodef-strategist--oagp-strategist--family-level-mcp-distribution-idea.openthing` so the idea surfaces during future family-level architecture discussion. Cross-spec — catdef-strategist holds substrate authority. Not memodef-spec's to drive.

== Open from this engagement, may need successor attention ==

1. **ccc-ninja-engineer memo in their inbox** (commit `pending`). Per receiver-commits-convention: their next session commits the typed memodef:Memo at `s:/projects/ccc-ninja/memos/inbox/2026-05-20-1955--...`. If the commit doesn't land within a reasonable window, surface to Director — not blocking, but the audit-trail incomplete until they commit.

2. **oagp-strategist memo at `s:/projects/oagp-org/memos/2026-05-21-1140--...`** (commit pending — receiver-side). Family-level MCP idea forward-work; not action-required.

3. **GitHub secret-scanner alerts on memodef-spec push** (`d50773e` commit). Two transcripts contain session UUIDs in `session_arc`. Director closed ccc-ninja-side alerts as false-positive 2026-05-21; same disposition for memodef-spec alerts when GitHub's scanner flags them. No spec text per KISS ruling above.

== Pattern observations worth carrying ==

- **Two-work-item engagement-split pattern Director used 2026-05-20.** When first-implementation surfaces both spec-side and tool-side concerns simultaneously: "1) spec updates, 2) engineer feedback" as parallel tracks in the same session. Keeps the two concern levels distinct. Re-use when shape recurs.

- **Three-folder-convention parallel composition.** `memos/{inbox,read,archive}/`, `notes/<role-id>/`, `transcripts/<role-id>/` now share working-repo-root placement and per-role-flat shape. Any future folder-convention proposal should compose with this triad rather than diverge.

- **POP-discipline lens: shape vs lifecycle.** New top-level type was warranted for transcripts (structurally different from memos: multi-participant, multi-timestamp, append-only, no action_required, no threading). Sentinel + folder was right for memos-to-file in v0.3 (same shape as memos, different workflow). Future enrichment proposals: first ask "is this shape, or is this lifecycle?" Shape = new type; lifecycle = sentinel/folder/optional-field.

- **First-implementation conformance verified BEFORE ratification is the strongest possible empirical confidence.** v0.2 and v0.3 had implementation arrive AFTER ratification; v0.4 inverted the sequence. Worth attempting again when the shape allows it (capture tools, hosted-store integrations, anything where a real implementer is in flight during proposal authoring).

== Required reading list for next strategist session arc ==

In order of decreasing priority:

1. `CLAUDE.md` — operating discipline, hard edges, AI-maintainer-vs-strategist scope
2. This memo + the rest of `notes/memodef-strategist/` (when accumulated)
3. `decisions/proposal-2026-05-20-transcript-type-and-folder-convention.md` (this engagement's ratification record)
4. `decisions/v0.3.1-implementer-experience-clarifications.md` (precedent for first-implementation-folded-back pattern)
5. `SCHEMA.md` (current normative spec text)
6. `README.md` (current adopter-facing material)
7. Strategist memory file at `C:/Users/edsby/.claude/projects/C--Users-edsby/memory/project_memodef.md` (cross-session institutional memory)

== Bot identity and authority context ==

Filed by `memodef-strategist@memodef.org` (provisional pending governance ratification). Implementation commit by `memodef-maintainer@memodef.org` (also provisional). Director-authorized scope expansion has had the same session arc play both hats with hat-distinction preserved in commit authorship — precedent from body_ref v0.2, memos-to-file v0.3, now v0.4 transcripts. When formally provisioned memodef.org governance exists, the strategist roledef may be derived from `senior-open-standards-strategist`; until then, the catdef-family bootstrap session arc plays this role per `decisions/bootstrap-deviation.md`.
