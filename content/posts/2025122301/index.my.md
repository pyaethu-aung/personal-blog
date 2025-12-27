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

### 2. Setup and Caching
`ubuntu-latest` ပေါ်မှာ Go `v1.25.5` ကို setup လုပ်ပြီး run ပါတယ်။ Caching အပိုင်းကတော့ အရင် [post]({{< relref path="2025121301/index.my.md" lang="my" >}}) မှာ ပြောခဲ့သလိုပဲ `go.sum` file မှာ အပြောင်းအလဲမရှိသ၍ Go modules တွေကို download မလုပ်ဘဲ cache ကနေ ပြန်ယူသုံးတဲ့အတွက် workflow ကြာချိန်ကို သိသိသာသာ လျှော့ချပေးပါတယ်။

### 3. Running Tests
ဒီ workflow မှာ test command တွေကို run ဖို့အတွက် `Makefile` ကို သုံးထားပါတယ်။ ဒါကြောင့် ကိုယ့် project ရဲ့ Makefile ထဲမှာ သက်ဆိုင်ရာ command တွေ ရှိနေဖို့တော့လိုပါမယ်။
1. `make test`: ဒါက ပုံမှန် unit test တွေကို run ဖို့အတွက်ပါ။ ဥပမာ `go test ./...` လိုမျိုး။
2. `make test-coverage`: ဒါကတော့ test coverage တွက်ဖို့နဲ့ report file (`coverage.out`) ထွက်လာအောင် run တာပါ။

### 4. Uploading Test Coverage
```yaml
- name: Upload coverage to Codecov
  uses: codecov/codecov-action@v5
  with:
    files: ./coverage.out
    flags: unittests
    fail_ci_if_error: false
```
Test coverage တွက်ပြီးရင် ရလာတဲ့ result ကို visually ကြည့်လို့ရအောင် [Codecov](https://about.codecov.io/) ကို upload တင်ပါတယ်။ `files: ./coverage.out` ဆိုပြီး step #4 မှာ ထွက်လာတဲ့ report file ကို ညွှန်းပေးထားပါတယ်။ `fail_ci_if_error: false` ကတော့ တခုခုကြောင့် codecov upload တင်တာ error တက်ခဲ့ရင်လည်း အဓိက test run တာ အောင်မြင်တယ်ဆိုရင် CI workflow ကို မ fail စေချင်လို့ပါ။

## နိဂုံး
ဒီ test workflow ထည့်ထားခြင်းအားဖြင့် PR တင်တဲ့အခါမှာ ကိုယ့် code ကြောင့် ရှိပြီးသား functionality တွေ ပျက်မသွားဘူးဆိုတာတဲ့ ယုံကြည်မှုမျိုး ရနိုင်မှာပါ။။ Test coverage report ကိုကြည့်ပြီးတော့လည်း ကိုယ့် project ရဲ့ ဘယ်အပိုင်းတွေမှာ unit test လိုနေသေးလဲဆိုတာ သိနိုင်ပါတယ်။ CI/CD pipeline မှာ testing က မပါမဖြစ်မို့လို့ ထည့်သုံးကြည့်ကြဖို့ တိုက်တွန်းချင်ပါတယ်။
