+++
title = "Go Linter With GitHub Actions"
date = '2025-12-13T22:59:00+07:00'
draft = true
lang = "en"
slug = "go-linter-with-github-actions"
tags = ["github-actions", "go"]
+++

It's been almost a year since my last blog post, which was back in January. I've been busy with personal things and also just feeling lazy.

Recently, I decided to try out [Bubble Tea](https://github.com/charmbracelet/bubbletea) and also wanted to write a small CLI app to calculate personal income tax in Myanmar. That led to the [PIT Calculator CLI app](https://github.com/pyaethu-aung/myanmar-pit-calculator). You can try it with both a simple CLI interface and a TUI interface. With the help of Copilot, I managed to write a proper README, so I encourage you to check it out if you have a chance. I still need to add features for life insurance, other tax exemptions, bonuses, and other taxable income types.

## Intro
In software development, code quality and consistency are important. In team projects, everyone writes code in their own style. This can lead to hard-to-read code, bugs, unreachable code, and inconsistent style. To help prevent these problems, especially when in PRs and merging code into stable branches, linting is important for any software projects.

In this post, I explain the GitHub Actions workflow I use for linting in the [PIT Calculator CLI app](https://github.com/pyaethu-aung/myanmar-pit-calculator). I recently started using GitHub Actions for my personal projects, except for Hugo deployment workflows.
