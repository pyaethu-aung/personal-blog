+++
title = "UUID Generator With AI Agent"
date = '2026-01-09T23:44:00+07:00'
draft = true
lang = "en"
slug = "uuid-generator-with-ai-agent"
tags = ["uuid", "react", "vite", "ai", "github-copilot"]
+++

## Intro
With the rise of AI assistants, the software development landscape has changed significantly. Tools like GitHub Copilot can now handle everything from simple CRUD apps to complex logic implementation. Developers can now focus more on high-level requirements.

In this post, I want to share about the [UUID Generator](https://github.com/pyaethu-aung/uuid-generator) project that I built using GitHub Copilot.

## UUID Generator Features
[UUID Generator](https://pyaethu-aung.github.io/uuid-generator/) is built with React 19, Vite, and Tailwind CSS. The main features are:
1. **UUID Version Support**: Supports V1 (timestamp-based), V4 (random), and V7 (time-ordered).
2. **Batch Generation**: Can generate up to 200 UUIDs at a time. The UI is optimized to preview only 20 items for performance.
3. **Smart Controls**: Batch size and preview count can be controlled with a unified slider.
4. **Copy & Download**: Supports clipboard copy and text file download (with timestamped filename).
5. **Formatting**: Supports uppercase, removing hyphens, and wrapping braces.
