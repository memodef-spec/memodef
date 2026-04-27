# templates/

Canonical memo templates demonstrating common patterns. Adopters authoring an operational memo SHOULD start from the closest matching template, fill in the placeholders, save the result to the recipient's `./memos/` inbox per their org's memo-location convention.

## What's a template

A template is a `memodef:Memo` artifact (or `memodef:Library` index) whose fields are populated with canonical placeholders rather than concrete values. Adopters copy the file, replace placeholders with real values, and save under the file naming convention (`YYYY-MM-DD-HHMM--<from>--<to>--<short-subject>.openthing`).

## What's NOT here

Operational memos. Those live in their recipient orgs' per-position working repos per the org-declared memo-location convention (e.g., catdef-org's convention is `<recipient_position.x.position.working_directory>/memos/`).

## Status

**Empty.** v0.1 bootstrap. The first template (handoff-memo, demonstrating the action-required handoff pattern) is forthcoming as PR #1.
