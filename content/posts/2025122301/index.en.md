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

## Breaking Down
### 1. Staring `on:`
Similar to the linter workflow, this workflow runs whenever code is pushed to the `main` branch or a pull request is created. It ensure the workflow only runs when necessary, Go source files (`cmd/**`, `pkg/**`), build-related and dependency files (`Makefile`, `go.mod`, `go.sum`), saving GitHub Actions minutes.

### 2. Setup and Caching
This job runs on `ubuntu-latest` and sets up Go `v1.25.5`. As mentioned in the previous [post]({{< relref path="2025121301/index.my.md" lang="my" >}}), the caching step is a key. It reuses the Go modules and build cache as long as `go.sum` hasn't changed, significantly reducing the workflow execution time.

### 3. Running Tests
This workflow relies on the `Makefile` to execute the test commands. You'll need to have the corresponding commands defined in your project's `Makefile`:
1. `make test`: Runs standard unit tests (e.g., `go test ./...`).
2. `make test-coverage`: Runs tests and generates a coverage report file (`coverage.out`).

### 4. Uploading Test Coverage
```yaml
- name: Upload coverage to Codecov
  uses: codecov/codecov-action@v5
  with:
    files: ./coverage.out
    flags: unittests
    fail_ci_if_error: false
```
After the tests run, it upload the results to [Codecov](https://about.codecov.io/) to visualize the coverage result. It point to the `coverage.out` file generated in the step #4 with `files: ./coverage.out`. Setting `fail_ci_if_error: false` ensures that if there's an issue uploading to Codecov, the entire CI workflow doesn't fail as long as the tests themselves passed.

## Conclusion
By adding this test workflow, you can have more confidence that your code won't break existing features when you submit a PR. Looking at the test coverage report also helps you identify which parts of your project still need unit tests. Testing is an indispensable part of any CI/CD pipeline, and I highly recommend setting it up for your projects.
