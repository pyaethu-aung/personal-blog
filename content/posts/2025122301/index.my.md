+++
title = "Unit Test With GitHub Actions"
date = '2025-12-23T23:54:00+07:00'
draft = true
lang = "my"
slug = "unit-test-with-github-actions"
tags = ["github-actions", "go"]
+++

## နိဒါန်း
ပြီးခဲ့တဲ့ [post]({{< relref path="2025121301/index.my.md" lang="my" >}}) မှာ Go project တွေအတွက် linter workflow အကြောင်း ပြောပြခဲ့ပြီးပါပြီ။ Code quality ကောင်းဖို့အတွက် linting က အရေးကြီးသလိုပဲ၊ ကိုယ်ရေးထားတဲ့ code က မျှော်မှန်းထားတဲ့အတိုင်း အလုပ်လုပ်ရဲ့လား၊ feature အသစ်တွေထည့်လိုက်ရင် အဟောင်းတွေကရော အလုပ်လုပ်သေးရဲ့လားဆိုတာ သေချာဖို့အတွက် testing ကလည်း မရှိမဖြစ် လိုအပ်ပါတယ်။

ကျွန်တော် ဒီ post မှာတော့ လက်ရှိ project တွေမှာ သုံးနေတဲ့ unit testing အတွက် GitHub Actions workflow အကြောင်း ပြောပြချင်ပါတယ်။

## Action File For Unit Tests
ဒါကတော့ Go project တွေမှာ unit tests run ဖို့နဲ့ code coverage တွက်ဖို့အတွက် ကျွန်တော်သုံးနေတဲ့ workflow file ပါ။
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
Linter workflow လိုပါပဲ၊ ဒီ workflow က main branch ကို code push လုပ်တဲ့အခါ ဒါမှမဟုတ် pull request တင်တဲ့အခါတွေမှာ အလုပ်လုပ်ပါလိမ့်မယ်။ Code နဲ့ဆိုင်တဲ့ file တွေ (`cmd/**`၊ `pkg/**` စသဖြင့်) နဲ့ `build`၊ `dependency` အပိုင်းနဲ့ဆိုင်တဲ့ `Makefile၊` `go.mod၊` `go.sum` တွေ ပြင်မှသာ ဒီ test workflow ကို run မှာဖြစ်လို့ မလိုအပ်ဘဲ workflow run နေတာမျိုး မဖြစ်အောင် ကာကွယ်ပေးပါတယ်။
