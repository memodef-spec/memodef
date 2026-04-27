# memodef Schema — v0.1.0

This document defines the structure and validation rules for **memodef artifacts**: portable, machine-readable memo-messages exchanged between positions in an organization, expressed as catdef-compliant `.openthing` files.

A memodef artifact captures one memo:

- A reader (human or AI) can identify the sender, recipient, subject, send timestamp, and content
- The artifact can be machine-routed (filtered by recipient id, threaded by `in_reply_to`, archived by date)
- Memos are interoperable with [orgdef](https://github.com/orgdef-spec/orgdef) (sender and recipient ids reference position ids in an orgdef:Organization)
- The memo can be transmitted via any transport (filesystem, MCP-as-notification, file-watcher, RSS, SMTP) without changing the memo content

This schema is the **load-bearing artifact** for memodef.

---

## Conformance language

This document uses RFC 2119 keywords:

- **MUST** / **MUST NOT** — required for conformance
- **SHOULD** / **SHOULD NOT** — recommended; well-designed memos follow these
- **MAY** — permitted; an author's choice

The schema also uses catdef's `x.` extension namespace pattern for adopter-defined extensions.

---

## Substrate: catdef

A memodef artifact is a catdef `.openthing` document. catdef provides the structural format; memodef defines the semantic shape inside that format. Every memodef artifact MUST:

- Be a valid catdef document (parseable as JSON conforming to catdef structural rules)
- Declare the catdef version it stamps under (`catdef` field)
- Declare the memodef schema version it conforms to (`memodef` field)
- Have its `type` set to one of the memodef-namespaced types (`memodef:Memo` for a single memo; `memodef:Library` for a catalog of memos or templates)

Beyond these constraints, memodef inherits catdef's full semantic toolkit (field types, polymorphic translatable fields, subcat enrichment, extension namespace, forward-compatibility rules).

---

## File extensions

memodef artifacts use the catdef substrate's universal formats:

```
2026-04-26-2030--roledef-strategist--catdef-canonical-implementor--pr5-fix.openthing  (single memo)
broadcasts.opencatalog                                                                  (memo catalog)
```

---

## File naming convention (RECOMMENDED)

Operational memos SHOULD use this filename pattern:

```
YYYY-MM-DD-HHMM--<from>--<to>--<short-subject>.openthing
```

This makes inboxes scannable by date + sender + recipient + subject without opening each file. The convention is recommended, not required — adopters MAY use other naming schemes if their inbox tooling expects them.

---

## Top-level structure: `memodef:Memo`

```json
{
  "catdef": "1.4",
  "memodef": "0.1.0",
  "type": "memodef:Memo",

  "from": "<sender position id>",
  "to": "<recipient position id, or 'all'>",
  "subject": "<short subject line>",
  "sent": "<ISO 8601 timestamp>",
  "body": "<markdown content>",

  "in_reply_to": "<predecessor filename, optional>",
  "action_required": <boolean, optional>,
  "thread_id": "<opaque identifier, optional>",

  "metadata": {...},

  "x.<domain>.<identifier>": ...
}
```

---

## Required fields (MUST) — `memodef:Memo`

### `catdef` (string, semver)

The catdef version the memo stamps under.

```json
"catdef": "1.4"
```

### `memodef` (string, semver)

The memodef schema version this memo conforms to.

```json
"memodef": "0.1.0"
```

### `type` (string, fixed value)

MUST be exactly the string `"memodef:Memo"` for a single memo, or `"memodef:Library"` for a catalog of memos or templates.

```json
"type": "memodef:Memo"
```

### `from` (string)

The sender's position id. The position id resolves in the org's `orgdef:Organization` artifact (positions array). For memos sent across orgs, the value MAY be qualified (e.g., `"catdef-org/roledef-strategist"`) — qualification syntax is not normatively defined in v0.1; adopters who need cross-org sending SHOULD declare their qualification convention in the org's orgdef artifact.

```json
"from": "roledef-strategist"
```

### `to` (string)

The recipient's position id, OR the literal string `"all"` for org-wide broadcasts. Same resolution rules as `from`.

When `to` is `"all"`: per the catdef-org convention, `"all"` addressing requires the sender to write the memo to every applicable position's inbox (which means having git/filesystem write access to all those repos). The friction is a feature — naturally limits org-wide broadcasts to high-authority senders.

```json
"to": "catdef-canonical-implementor"
```

```json
"to": "all"
```

### `subject` (string)

Short subject line. SHOULD be scannable in an inbox listing (suggested ≤80 characters, hard-cap at no specific length).

```json
"subject": "PR #5 in catdef.org — switch outer template literal to String.raw"
```

### `sent` (string, ISO 8601)

Timestamp when the memo was sent. ISO 8601 format with timezone (Zulu/UTC preferred).

```json
"sent": "2026-04-26T20:30:00Z"
```

### `body` (RichText)

The memo content. Markdown. SHOULD use the plain-text `== Section ==` convention for cross-session readability — survives copy-paste through transport mechanisms that may strip formatting.

```json
"body": "== Background ==\n\n...\n\n== The ask ==\n\n..."
```

---

## Recommended fields (SHOULD)

### `in_reply_to` (string)

Filename of the predecessor memo when this memo is a reply. Used for threading. The filename SHOULD be the predecessor's bare filename (no path), since the reply lives in a different working repo than the predecessor in cross-position cases.

```json
"in_reply_to": "2026-04-26-2030--roledef-strategist--catdef-canonical-implementor--pr5-string-raw-template-literal.openthing"
```

### `action_required` (boolean)

Whether the recipient is expected to take action, or this is informational only. Defaults to `false` if absent.

```json
"action_required": true
```

### `thread_id` (string)

Opaque identifier grouping related memos across replies. Useful for tools that visualize memo conversations. v0.1 doesn't normatively define a thread-id format — adopters MAY use any identifier (UUID, predecessor's filename, manually-assigned slug). `in_reply_to` chains provide implicit threading; `thread_id` makes it explicit when adopters want it.

```json
"thread_id": "pr5-template-literal-fix"
```

### `metadata` (object)

Authorship, licensing, sender/recipient alternative identifiers, attachments references, retention policy. v0.1 leaves this loosely typed.

```json
"metadata": {
  "sender_session_arc": "...",
  "attachments": [],
  "retention_policy": "permanent"
}
```

---

## Top-level structure: `memodef:Library`

A catalog of memo artifacts. Used for memo catalogs (e.g., a templates library, or an archived broadcast catalog). Same shape as catdef catalogs.

```json
{
  "catdef": "1.4",
  "memodef": "0.1.0",
  "type": "memodef:Library",
  "id": "<library-id>",
  "name": "<human-readable name>",
  "description": "...",
  "version": "<semver>",
  "data": {
    "items": [
      {
        "id": "<memo-or-template-id>",
        "name": "...",
        "version": "...",
        "description": "...",
        "file": "<path>",
        ...
      }
    ]
  },
  "metadata": {...}
}
```

---

## Backward-compatible legacy form: `x.memo.*` extensions on a generic catdef thing

Memos written under the legacy `x.memo.*` extension form on a generic `catdef:Thing` (per the catdef-org `x.org.memo_extension_namespace` declaration before memodef bootstrapped) are valid catdef artifacts and remain readable by memodef-aware consumers per catdef's reader-lenient discipline.

Legacy form:

```json
{
  "catdef": "1.4",
  "type": "thing",
  "thing": { "template": "memo", "fields": {...} },
  "x.memo.from": "...",
  "x.memo.to": "...",
  "x.memo.subject": "...",
  "x.memo.sent": "...",
  "x.memo.in_reply_to": "...",
  "x.memo.action_required": false,
  "body": "..."
}
```

**Migration path:** new memos SHOULD use `type: "memodef:Memo"` with top-level fields. Legacy memos SHOULD NOT be retroactively rewritten (audit-trail integrity); they remain valid in their original form.

The mapping from legacy to typed:

| Legacy | Typed |
|---|---|
| `x.memo.from` | `from` |
| `x.memo.to` | `to` |
| `x.memo.subject` | `subject` |
| `x.memo.sent` | `sent` |
| `x.memo.in_reply_to` | `in_reply_to` |
| `x.memo.action_required` | `action_required` |
| `body` | `body` (unchanged) |

memodef-aware consumers (validators, renderers, inbox tools) SHOULD recognize both forms.

---

## Extension fields (`x.` namespace)

memodef artifacts MAY include any number of extension fields under the `x.<domain>.<identifier>` namespace. Per the catdef-family self-description SHOULD-rule, each extension SHOULD carry a `description` field for naive-agent handling.

Plausible extensions (illustrative, not normative):

```
"x.security.encryption": "..."
"x.security.signature": "..."
"x.legal.confidentiality": "internal-only"
"x.org.priority": "urgent"
"x.transport.notification_sent_at": "..."
"x.attachments.references": [...]
```

---

## Reserved namespaces

The following namespaces are reserved by the memodef spec and MUST NOT be used as adopter extensions:

- `memodef:*` — reserved for memodef spec types and fields
- `catdef:*` — reserved by catdef substrate
- `x.memodef.*` — reserved for memodef-spec-coordinated extensions

Adopters use their own `x.<domain>.<identifier>` namespaces.

---

## Internal consistency rules

A valid `memodef:Memo` MUST:

1. Have all 5 required top-level fields (`from`, `to`, `subject`, `sent`, `body`) present and non-empty
2. Have `sent` parseable as an ISO 8601 timestamp
3. Have `type` exactly `"memodef:Memo"`
4. Have `catdef` and `memodef` version stamps consistent with the features used

A valid `memodef:Memo` SHOULD:

1. Use the recommended file naming convention (`YYYY-MM-DD-HHMM--<from>--<to>--<short-subject>.openthing`)
2. Place `body` as Markdown using the `== Section ==` convention for cross-transport readability
3. When a reply, populate `in_reply_to` with the predecessor's bare filename
4. Include `metadata.sender_session_arc` (or equivalent) when the sender is an AI session whose arc identifier is meaningful for audit

---

## Future considerations (not in v0.1)

These have come up in design discussion but are deferred to v0.2+:

- **`memodef:Thread` type** for explicit thread management when `in_reply_to` chains aren't enough
- **`memodef:Broadcast` type** for one-to-many memos that explicitly want fan-out tracking (vs. just `to: "all"` write-to-each-inbox semantics)
- **Cross-org `from`/`to` qualification syntax** (e.g., `"catdef-org/roledef-strategist"`) — currently informal
- **Attachments structured field** — currently via `x.attachments.*` extensions
- **Encryption / signing structured fields** — currently via `x.security.*` extensions
- **Notification-layer spec** (separate? part of memodef?) — the protocols transport mechanisms use to notify recipients of new memos (file-watcher, MCP-as-notification, RSS feed format, SMTP envelope, etc.)
- **Retention / archival policy structured fields** — currently via `metadata.retention_policy` informal
- **Read-receipt / delivery-confirmation patterns** — currently informal

These belong to v0.2+ and will be triaged as memodef matures.

---

## Versioning

memodef follows semver:

- **Patch** (0.1.X): clarifications, editorial fixes, no behavior changes
- **Minor** (0.X.0): new optional fields, new standard relationship types, backward-compatible additions
- **Major** (X.0.0): breaking changes (removed required fields, changed semantics)

The schema version is independent from any individual memo's `metadata.version` (most memos won't have one).

---

## Examples

### Minimal `memodef:Memo`

```json
{
  "catdef": "1.4",
  "memodef": "0.1.0",
  "type": "memodef:Memo",
  "from": "alice",
  "to": "bob",
  "subject": "Quick question about Q3",
  "sent": "2026-04-26T14:00:00Z",
  "body": "Bob — what's the Q3 deadline?\n\n— Alice"
}
```

### Reply with threading

```json
{
  "catdef": "1.4",
  "memodef": "0.1.0",
  "type": "memodef:Memo",
  "from": "bob",
  "to": "alice",
  "subject": "Re: Quick question about Q3",
  "sent": "2026-04-26T14:30:00Z",
  "in_reply_to": "2026-04-26-1400--alice--bob--quick-question-about-q3.openthing",
  "thread_id": "q3-deadline-thread",
  "body": "Alice — Sept 30, end of business. Filing the calendar invite now.\n\n— Bob"
}
```

### Action-required handoff (handoff-memo template shape)

```json
{
  "catdef": "1.4",
  "memodef": "0.1.0",
  "type": "memodef:Memo",
  "from": "roledef-strategist",
  "to": "catdef-canonical-implementor",
  "subject": "PR #5 in catdef.org — switch outer template literal to String.raw",
  "sent": "2026-04-26T20:30:00Z",
  "action_required": true,
  "body": "== Background ==\n\n[...]\n\n== The ask ==\n\n[...]\n\n== Don'ts ==\n\n[...]\n\n== Done ==\n\n[...]"
}
```

### Org-wide broadcast

```json
{
  "catdef": "1.4",
  "memodef": "0.1.0",
  "type": "memodef:Memo",
  "from": "orgdef-strategist",
  "to": "all",
  "subject": "Memo convention v1.0 — please use ./memos/ in your own working repo",
  "sent": "2026-04-26T20:00:00Z",
  "action_required": false,
  "body": "[announcement body]"
}
```

---

## Status

**v0.1.0 — bootstrap.** This schema formalizes the `x.memo.*` extension namespace established empirically by catdef-org (orgdef PR #1, 2026-04-26). Stable shape expected by v0.1.x; substantive changes go through the proposal workflow per CONTRIBUTING.md.
