# htmlllm

**htmlllm** is a **human-first** kind-of single-file kind-of writing-harness.

## Why
If you're tired of reviewing what agents write, turn the tables: let agents review what you write.

Writing clearly, and most importantly, understanding, takes time and no amount of innovation in LLMs will change that.

To be a bit tongue-in-cheek: don't get caught again saying "I don't know, the agent wrote it". Write what you know, ask questions and let agents help with a more thorough understanding. Repeat.

## Features
- **simple**: no server, database, API or setup, just a simple markdown-based block editor communicating with a session on your machine
- **agent is a reviewer**: posts comments on paragraphs, replies to your comments
- **brief** vs. **explanatory** mode: the style of agent's comments
- **basic export**: click Export, it opens a panel with complete markdown (content only, not revision comments); copy it and paste it where you need it

## Requirements
- chromium-based browser (Chrome, Brave) or Edge with **File System Access API** enabled
- Claude with Monitor tool (built-in)

## Mechanistic view

**html file is the shell, json file is the yolk**.

When you perform an action (add a paragraph, edit paragraph, create a comment, reply to a comment, resolve a comment), it is written to the json file. Agent has a monitor on that file and watches for changes. Once it detects the change, it parses the change and decides what to post as a comment (or not to post a thing).

## Quick start
- clone this repository; navigate to the directory
- make a copy of `copyme.html`, give it a descriptive name
- open a file in **Chrome/Edge**
- click on **Start a new doc** and create an associated json file; give it a descriptive name
- start a new interactive agent session in the directory where `AGENTS.md` is (current directory if you didn't change it)
- send it a message ("read AGENTS.md") and agent will ask you which files to watch; choose the newly created pair
- start what you do best

You can also run this in a separate project if you want to use that specific project's memory. In that case, copy the `copyme.html`, `theme.css` (if you want theming) and point an agent to the htmlllm's AGENTS.md so it knows what to do. If you plan on using it continuously in another repository, it's advisable to integrate htmlllm's AGENTS.md / CLAUDE.md into that project's AGENTS.md / CLAUDE.md.

## Caveats
As you can assume from the mechanistic view above, this kind of work is slow, or at least slower than you regular agentic sessions. A json file is a level of indirection.

Can it use an API instead of local session? Yes, probably, but my focus was on almost zero-setup.

This was tested mainly with **Claude**. With an agent that lacks a monitor-like tool you may have to get creative, or just use something else.
