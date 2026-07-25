# htmlllm — agent instructions

This is a collaborative doc: the user writes paragraphs and comments in an
HTML file (opened directly via `file://` in Chrome/Edge — no server), and an
agent (you) watches its paired `.json` data file for changes and reacts with
comments/replies.

Nothing here is specific to any one agent tool. The protocol is plain file
I/O: read a JSON file, look at a couple of flags, write back. Any agent that
can read/write files and run shell commands can follow it.

## This directory can hold multiple docs

`copyme.html` is the pristine template — the user duplicates it (e.g. to
`newdoc.html`) to start a new doc, then uses its "New doc…" button to create
and connect a fresh `.json` data file (or "Connect existing doc.json" to
reopen one). `doc.html` may itself just be one particular instance. **Do not
assume `doc.json` is the file to watch** — this directory may contain several
independent doc/json pairs at once, each unrelated to the others (each HTML
file remembers its own connected `.json` via a browser-side handle keyed to
its own path — they never share state).

**At the start of a session, ask the user which `.json` file is the active
working copy** before setting up anything. Only proceed to "Watching for
changes" below once you know the specific path.

## Files (per doc instance)

- `<name>.html` — the app (a copy of `copyme.html`, possibly customized).
  Static shell + JS. Uses the File System Access API to read/write its
  connected `.json` directly. No server, no build step.
- `<name>.json` — the single source of truth for that doc. Both the browser
  and you read and write this file directly. There is no other channel
  between you and the page — everything goes through this file.
- `theme.css` — a plain CSS file, linked via `<link rel="stylesheet">` in
  every doc's `<head>` (no picker, no permissions — just a normal relative
  file load). Starts as a live copy of the default color variables; the user
  edits it directly and reloads to see changes. Shared across every doc made
  from `copyme.html` since they all link the same relative path. It's not
  doc content — never react to it or treat changes to it as something to
  comment on.

## Schema

```
{
  rev: number,                 // bumped on every real content mutation (not status pings)
  paragraphs: [
    {
      id, text (markdown), createdAt, updatedAt,
      pendingChange: null | { type: "new" | "edited", prevText: string | null, since: number },
      comments: [ commentNode, ... ]
    }
  ],
  settings: { tone: "brief" | "explanatory" },
  lastEvent: { type, paragraphId, ..., by: "browser" | "claude", at },  // last real content mutation
  agentStatus: { text, state: "idle" | "working", at }                  // optional live status ticker
}
```

`commentNode`: `{ id, author: "user" | <your agent name>, text (markdown), createdAt, needsAgent: bool, resolved: bool (meaningful on top-level comments only), replies: [commentNode, ...] }`

Identify yourself honestly: `"user"` is the one special-cased value (rendered
as "You"); anything else in `author` or `lastEvent.by` is treated as an
agent and displayed under its own name, capitalized — so write your actual
name (`"codex"`, `"claude"`, `"gemini"`, whatever you are), not a
hardcoded/copied value from an earlier session. If a doc's history already
has comments from a different agent, that's fine — each comment keeps
whichever name authored it.

## Watching for changes

Once you know which `.json` file is the working copy (see above — ask first,
don't guess), you need something that detects when its mtime changes and
tells you. The core check, in plain bash:

```bash
f="/absolute/path/to/<name>.json"   # the file the user told you to watch
last=$(stat -f %m "$f" 2>/dev/null || echo 0)
cur=$(stat -f %m "$f" 2>/dev/null || echo 0)
if [ "$cur" != "$last" ]; then
  ev=$(python3 -c "import json
try:
    d=json.load(open('$f'))
    e=d.get('lastEvent')
    print(json.dumps(e) if e else 'null')
except Exception as exc:
    print('parse-error: %s' % exc)
" 2>/dev/null)
  echo "$(basename "$f") changed: $ev"
fi
```

How you turn that into an ongoing watch depends entirely on what your tool
gives you — there's no universal primitive for this across agent CLIs:

- **If your tool has a persistent background-process/notification
  capability** (e.g. Claude Code's `Monitor` tool), wrap the check above in
  a `while true; do sleep 1.5; ...; done` loop and run it that way — each
  detected change becomes a notification pushed back into this same
  conversation, without you polling manually.
- **If it doesn't** (as of this writing, Codex CLI has no equivalent —
  it's an open feature request, not a shipped capability), you have two
  options: run the loop as a bounded, blocking command (e.g. loop with a
  timeout, exit as soon as a change is detected or the timeout is hit),
  react, then re-enter the same blocking watch yourself — or be invoked
  periodically by something external (cron, a wrapper script, the user
  re-prompting you) rather than watching continuously within one turn.

Either way: every notification could be a real user action, or your own
previous write echoing back (the watcher can't tell — mtime changes either
way). Check `lastEvent.by` and the actual `pendingChange`/`needsAgent` state
in the file before assuming there's something to react to.

## When to react

Read the file. React to:

- Any paragraph with `pendingChange !== null` — a new or edited paragraph.
  If `type: "edited"`, diff the current text against `prevText` and do a
  genuine cross-check (does the edit contradict or resolve prior discussion?)
  rather than reacting generically.
- Any comment, recursed through `replies`, with `needsAgent: true` — unless
  it sits under a top-level comment with `resolved: true` (resolved threads
  are closed; don't react inside them even if a stale `needsAgent: true`
  lingers on a reply).

## How to react

Always read-modify-write against the **freshest** copy of the file — re-read
right before writing, don't rely on a stale in-memory copy — to avoid
clobbering a concurrent edit from the browser. Typical reaction:

1. Optionally set `agentStatus` to a short "working" message (no `rev` bump
   needed — this is a pure heartbeat, not a content change).
2. Do the actual work (read code, fetch a source, think it through).
3. Append a comment node (`author: "claude"`, `needsAgent: false`,
   `resolved: false`) to the relevant paragraph, or nest a reply under the
   comment that triggered you.
4. Clear the trigger: set `pendingChange: null` on the paragraph and/or
   `needsAgent: false` on the comment you answered.
5. Bump `rev` by 1.
6. Clear `agentStatus` back to `{ state: "idle", text: "Idle — watching for changes", at }`.

## Conventions to respect

- **Tone**: check `settings.tone` (`"brief"` or `"explanatory"`) and match
  your reply length/depth to it.
- **Never resolve a thread yourself** — `resolved` is a user action. You may
  suggest resolving in a reply, but don't set it.
- **Don't edit paragraph text directly** unless the user has explicitly asked
  for that in the current conversation — paragraphs carry no authorship
  marker, so an agent-written edit would be indistinguishable from the
  user's own writing. Comment instead, or ask first.
- **Don't delete paragraphs** on your own initiative — there's a 🗑 delete
  button in the UI, but it's a user action, same as resolving.
- **Markdown** is supported in both paragraphs and comments: `#`/`##`/`###`
  headers, `**bold**`, `*italic*` (no `_underscore_` form — avoids breaking
  `snake_case` identifiers), `~~strike~~`, `++underline++`, `` `code` ``,
  `[text](url)`, `![alt](path)` for images (external files only, relative
  path — never base64; see the "Files" note on why `.json` stays lean), and
  reference-style `[text][id]` / shorthand `[id]` with a `[id]: url`
  definition line — scoped per-paragraph (each block renders independently;
  there's no whole-document markdown pass yet).
- Keep replies grounded — verify things (read the actual file, fetch the
  actual source) rather than answering from assumption. This document's own
  content is largely about exactly that distinction.

## Starting a fresh session

Point a new agent session (Claude Code, Codex CLI, or any other tool that
reads this file) at this directory — most agent CLIs auto-load `AGENTS.md`.
Ask the user which `.json` file is the active working copy for this session,
then start watching that specific file using whatever mechanism your tool
provides (see "Watching for changes" above), and you're live.
