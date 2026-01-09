+++
title = "UUID Generator With AI Agent"
date = '2026-01-09T23:44:00+07:00'
draft = true
lang = "en"
slug = "uuid-generator-with-ai-agent"
tags = ["uuid", "react", "vite", "ai", "github-copilot"]
+++

[မြန်မာဘာသာဖြင့် ဖတ်ရှုရန်]({{< relref path="2026010701/index.my.md" lang="my" >}})

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

## Breaking Down
### 1. AI-First Development
The unique aspect of this project is that almost all core implementation was written by **GitHub Copilot** (GPT-5.1-Codex). Human inputs were limited to high-level requirements, code review, and testing guidance. React architecture, Tailwind styling, business logic, and unit tests were fully implemented by GitHub Copilot.

### 2. Prompt Evolution
The workflow didn't follow the traditional planning -> coding pattern but rather a **conversational iteration** approach. If you look at the prompt history documented in the `README`, you can see how features were built incrementally:
1. Initial Interface styling
2. Version selector logic
3. Mobile responsive badges
4. UX interactions (copy feedback)
5. Scaling to 200 items
6. Unified slider control
7. Information architecture refinement

### 3. Technical Highlights
The implementation quality provided by Copilot is quite impressive.
- **Custom Hooks**: Logic is separated into `useTheme` and `useUuidGenerator`, making the code clean and easy to test.
- **Web Crypto API**: Uses `crypto.randomUUID()` via the Web Crypto API instead of `Math.random()` for true randomness.
- **Optimized Rendering**: Includes logic to render only items in the viewport instead of rendering all 200 items.

## Conclusion
This project is a good example for testing the AI-assisted development workflow. It allowed me as a developer to focus more on architectural vision and quality assurance rather than implementation details. You can explore the project source code on [GitHub](https://github.com/pyaethu-aung/uuid-generator) and try out the [Demo](https://pyaethu-aung.github.io/uuid-generator/).
