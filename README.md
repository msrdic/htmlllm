# htmlllm

**htmlllm** is a **human-first** kind-of single-file kind-of writing-harness.

## Why
If you're tired of reviewing what agents write, flip the tables: let agents review what you write.

## How it works
- clone this repository; navigate to the directory
- make a copy of `copyme.html`, give it a descriptive name
- open a file in **Chrome/Edge**
- click on **Start a new doc** and create an associated json file; give it a descriptive name
- start a new interactive agent session in the directory where `CLAUDE.md` is (current directory if you didn't change it)
- send it a message ("read CLAUDE.md") and agent will ask you which files to watch; choose the newly created pair
- start what you do best

## Features
- **simple**: no server, database or API, just a simple markdown-based block editor communicating with a session on your machine
- **agent is a reviewer**: posts comments on paragraphs, replies to your comments
- **brief** vs. **explanatory** mode: the style of agent's comments
- **basic export**: click Export, it opens a panel with complete markdown (content only, not revision comments); copy it and paste it where you need it

## Mechanistic view

**html file is the shell, json file is the yolk**.

<img width="300" height="100" alt="mechanistic-view" src="https://github.com/user-attachments/assets/a9b6b487-1405-48fc-a8c0-aa64c2faee5d" />

When you perform an action (add a paragraph, edit paragraph, create a comment, reply to a comment, resolve a comment), it is written to the json file. Agent has a monitor on that file and watches for changes. Once it detects the change, it parses the change and decides what to post as a comment (or not to post a thing).

## Caveats
As you can assume from the mechanistic view above, this kind of work is slow, or at least slower than you regular agentic sessions. A json file is a level of indirection.

It is opinionated. Writing clearly, and most importantly, understanding, takes time and no amount of innovation in LLMs will change that.

To be a bit tongue-in-cheek: don't get caught again saying "I don't know, the agent wrote it". Write what you know, ask questions and let agents help with a more thorough understanding. Repeat.

It only works in Chrome (or Chromium-based browsers) and Edge since it uses the File System Access API.
