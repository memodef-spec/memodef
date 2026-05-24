# Claude Code Transcript

---

## 🧑 User <sub>5/21/2026, 9:00:00 AM</sub>

Hi Claude — I'd like to draft a canonical transcript fixture for memodef v0.4. What's the minimal viable structure that exercises the body_ref pattern with realistic formatting?

---

## 🤖 Assistant *(claude-opus-4-7)* <sub>5/21/2026, 9:00:08 AM</sub>

The minimum is: speaker labels + timestamps + content. Markdown formatting (headers, lists, code blocks) demonstrates that the capture_format field's "markdown" value carries durable formatting through paste-flatten cycles. A short example with one tool invocation rounds out the fixture.

```
example tool call output preserved verbatim
```

That covers the shape adopters need to recognize.

---

## 🧑 User <sub>5/21/2026, 9:00:42 AM</sub>

Perfect — short enough to fit in a conformance fixture, long enough to exercise the markdown-survival property.

---
