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

## Action File For Unit Tests
This is the workflow file I use to run unit tests and calculate code coverage for Go projects.
```yaml
name: Unit Tests

on:
  push:
    branches: [main]
    paths:
      - "cmd/**"
      - "pkg/**"
      - "go.mod"
      - "go.sum"
      - "Makefile"
  pull_request:
    branches: [main]
    paths:
      - "cmd/**"
      - "pkg/**"
      - "go.mod"
      - "go.sum"
      - "Makefile"

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v6

      - name: Set up Go
        uses: actions/setup-go@v6
        with:
          go-version: "1.25.5"

      # Caching prevents redownloading dependencies on every run until
      # go.mod/go.sum changes
      - name: Cache Go modules
        uses: actions/cache@v5
        with:
          path: |
            ~/go/pkg/mod # Go modules cache
            ~/.cache/go-build # Go build cache
          key: ${{ runner.os }}-go-${{ hashFiles('**/go.sum') }}

      - name: Run unit tests
        run: make test

      - name: Run tests with coverage
        run: make test-coverage

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v5
        with:
          files: ./coverage.out
          flags: unittests
          fail_ci_if_error: false
```
