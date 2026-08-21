---
"@aoagents/ao-core": patch
---

Stop `ao send` from destroying live sessions and failing successful restores.
Two fixes in the send/restore path:

1. A merged PR marks the lifecycle terminal (`isTerminalSession` checks
   `pr.state === "merged"`) while the agent may still be alive and idle in
   `merged_waiting_decision`. Send treated that policy verdict as "session is
   not running" and force-restored — killing a healthy runtime
   mid-conversation. When terminality stems only from the merge, send now
   probes the runtime and agent process and delivers to the live session;
   genuinely dead sessions still restore.

2. `waitForRestoredSession` required the tmux pane's foreground command to
   equal the agent process name, but the launch-script wrapper makes
   `pane_current_command` report "bash" for a healthy wrapped agent — so every
   tmux restore-for-delivery timed out with "restored session did not become
   ready" even when the restore succeeded. The process probe now decides,
   with the foreground match kept only as a fallback signal. The equivalent
   check in `waitForInteractiveReadiness` (which made bootstrap sends wait out
   their full timeout) is removed for the same reason.
