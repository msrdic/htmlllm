@AGENTS.md

## Claude Code specifics

Use the `Monitor` tool (with `persistent: true`) to run the watch loop
described in AGENTS.md — it pushes each detected change back into this
conversation as a notification, so you don't need to poll manually or
re-enter the loop yourself.
