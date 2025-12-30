+++
title = "Unit Test With GitHub Actions"
date = '2025-12-29T23:19:00+07:00'
draft = true
lang = "en"
slug = "unit-test-with-github-actions"
tags = ["github-actions", "go"]
+++

## Intro
In my previous [post]({{< relref path="2025121301/index.my.md" lang="my" >}}), I talked about setting up a linter workflow for Go projects. While linting is essential for maintaining code quality, testing is equally important to ensure your code works as expected and to make sure newly added features don't break existing functionality.

In this post, I want to share the GitHub Actions workflow I use for unit testing in my current projects.
