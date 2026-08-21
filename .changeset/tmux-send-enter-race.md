---
"@aoagents/ao-plugin-runtime-tmux": patch
---

Fix silently dropped sendMessage submissions: the fixed 300ms delay before
pressing Enter loses the race against Claude Code's paste-chip ingestion for
messages around 1KB and up — the Enter is swallowed, the message sits typed
but unsubmitted, and ao reports success. The settle delay now scales with
message size, and after Enter the pane is captured to verify the input box
actually cleared, re-pressing Enter (bounded) if it did not. Verification is
best-effort: an unreadable pane counts as submitted, and a false "pending"
costs one extra Enter on an empty input box, which agent TUIs treat as a
no-op.
