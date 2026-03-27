+++
title = "Why claude-replay is useful for Claude Code Users"
date = '2026-03-24T23:00:00+07:00'
draft = false
lang = "en"
slug = "claude-replay-for-claude-code"
tags = ["claude", "claude code", "claude-replay", "tools"]
+++

[မြန်မာဘာသာဖြင့် ဖတ်ရှုရန်]({{< relref path="2026032401/index.my.md" lang="my" >}})

## Introduction

Recently, I have been using both Claude Code and Antigravity for work and personal projects. However, I encountered a problem: it's quite inconvenient to share the conversations from a session with others. Sending long logs or recording a screen capture isn't very practical.

While looking for a solution, I found a tool called [claude-replay](https://github.com/es617/claude-replay). **claude-replay** generates an interactive playback HTML file that allows you to watch AI chat sessions just like a video.

Currently, it only supports Claude Code, Cursor, and Codex CLI.

## Useful Commands
Some useful commands you should know:
- `claude-replay`: Launch the web editor to browse and view your current sessions.
- `claude-replay abc123def456 -o replay.html`: Pass a session ID to extract it and generate an Interactive playback HTML file.
- `claude-replay session.jsonl --no-thinking --no-tool-calls -o replay.html`: Hide AI thinking steps and tool calls if you do not want to include them.
- `claude-replay session.jsonl --turns 5-15 --speed 2.0 -o replay.html`: Play back only a specific part of the session and customize the playback speed.
- `claude-replay session.jsonl --theme dracula -o replay.html`: Change the playback UI color theme easily.

## Conclusion
For features like [Secret redaction](https://github.com/es617/claude-replay?tab=readme-ov-file#secret-redaction) and other extremely useful capabilities, as well as the ability to embed the single HTML file into another page, I highly recommend checking out the [claude-replay](https://github.com/es617/claude-replay) GitHub Repo and trying it out yourself!
