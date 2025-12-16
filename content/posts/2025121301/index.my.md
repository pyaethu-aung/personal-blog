+++
title = "Go Linter With GitHub Actions"
date = '2025-12-13T22:59:00+07:00'
draft = false
lang = "my"
slug = "go-linter-with-github-actions"
tags = ["github-actions", "go"]
+++

နောက်ဆုံး blog post ရေးခဲ့တာက ပြီးခဲ့တဲ့ဇန်နဝါရီလကဆိုတော့ ဘာမှမရေးဖြစ်ခဲ့တာ တနှစ်နီးပါးလောက်တောင် ရှိသွားခဲ့ပါပြီ။ ဒီကြားထဲမှာ ကိုယ်ရေးကိုယ်တာကိစ္စတွေနဲ့ အာရုံများနေတာရော ပျင်းနေတာရောပါတယ်။ ဒီရက်ပိုင်းမှ [Bubble Tea](https://github.com/charmbracelet/bubbletea) ကို စမ်းကြည့်ချင်တာရော personal income tax တွက်ဖို့ CLI app လေးတခုရေးချင်တာရောနဲ့ [PIT Calculator CLI app](https://github.com/pyaethu-aung/myanmar-pit-calculator) လေးလုပ်ဖြစ်တယ်။ ရိုးရိုး CLI interface နဲ့ TUI interface ဆိုပြီး နှစ်ခုနဲ့ စမ်းကြည့်လို့ရပါတယ်။ Copilot အကူအညီနဲ့ ပါ README သေချာရေးဖြစ်သွားတော့ အဆင်ပြေရင် စမ်းကြည့်ကြစေချင်ပါတယ်။ Life insurance နဲ့ တခြား အခွန်သက်သာခွင့်တွေရယ် bonus နဲ့ တခြား taxable income တွေအတွက်ရယ်တော့ ထပ်ထည့်ဖို့တော့ ကျန်ပါဦးမယ်။

## နိဒါန်း
Software development မှာ code quality နဲ့ consistency ဆိုတာ မဖြစ်မနေလိုအပ်တဲ့အရာတွေပါ။ Team နဲ့လုပ်ရတဲ့ project တွေမှာဆို coding style တယောက်တမျိုးစီရေးကြတာတွေကြောင့် ဖတ်ရခက်တာမျိုးတွေ၊ bug ဖြစ်စေနိုင်တဲ့ ပုံစံမျိုးတွေ မပါဖို့နဲ့ unreachable code တွေ မရှိနေဖို့ အတွက် PR နဲ့  stable branch ကို code merge တာတွေမှာ linting က အရေးကြီးပါတယ်။

အခု post မှာတော့ [PIT Calculator CLI app](https://github.com/pyaethu-aung/myanmar-pit-calculator) မှာသုံးထားတဲ့ linter အတွက် workflow အကြောင်းကွက်ပြီး ပြောပြချင်လို့ပါ။ အလုပ်မှာကတော့ သုံးဖြစ်ပေမည့် personal project မှာတော့ Hugo ရဲ့ deployment အတွက်ကလွဲရင် အခုမှစသုံးဖြစ်တာပါ။

## Workflow For Lint
ဒါက ကျွန်တော်လက်ရှိသုံးဖြစ်နေတဲ့ Go အတွက် linter workflow ပါ။
```yaml
name: Lint

permissions:
  contents: read

on:
  push:
    branches: [main]
    paths:
      - "cmd/**"
      - "pkg/**"
      - "go.mod"
      - "go.sum"
      - ".github/workflows/lint.yml"
  pull_request:
    branches: [main]
    paths:
      - "cmd/**"
      - "pkg/**"
      - "go.mod"
      - "go.sum"
      - ".github/workflows/lint.yml"

jobs:
  lint:
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

      # Ensures gofmt sees only actual Go code that belongs to your module
      - name: Run go fmt
        run: |
          FILES=$(gofmt -s -l $(go list -f '{{.Dir}}' ./...))
          if [ -n "$FILES" ]; then
            echo "❌ Formatting issues found:"
            echo "$FILES"
            exit 1
          fi
          echo "✅ Code formatting is correct"

      - name: Run go vet
        run: |
          go vet ./...
          echo "✅ Code analysis passed"

      - name: Check for imports
        run: |
          go mod tidy
          if [ -n "$(git diff --name-only go.mod go.sum)" ]; then
            echo "❌ go.mod/go.sum are not tidy. Run 'go mod tidy' to fix:"
            git diff
            exit 1
          fi
          echo "✅ Dependencies are tidy"
```

## Breaking Down
### 1. Staring `on:`
```yaml
on:
  push:
    branches: [main]
    paths:
      - "cmd/**"
      - "pkg/**"
      - "go.mod"
      - "go.sum"
      - ".github/workflows/lint.yml"
  pull_request:
    branches: [main]
    paths:
      - "cmd/**"
      - "pkg/**"
      - "go.mod"
      - "go.sum"
      - ".github/workflows/lint.yml"
```
ဒီ `on:` အပိုင်းက workflow ကို ဘယ်အချိန်မှာစအလုပ်လုပ်မယ်ဆိုတာကို သတ်မှတ်ထားတာပါ။
1. `push` နဲ့ `pull_request` နှစ်ခုလုံးမှာ အလုပ်လုပ်ပါမယ်။
2. `branches: [main]` ဆိုတာက `main` branch ကို code push တာ ဒါမှမဟုတ် `main` branch ကို PR တင်တာမျိုးလုပ်မှသာ အလုပ်လုပ်ပါလိမ့်မယ်။
3. `paths:` မှာ `cmd/**`၊ `pkg/**` အစရှိတဲ့ Go file တွေ ဒါမှမဟုတ် workflow file `lint.yml` ကို ပြင်မှသာ ဒီ workflow ကို run မှာဖြစ်လို့ လိုအပ်တဲ့အချိန်မှာပဲ run ပြီး GitHub Actions အတွက် ကုန်ကျစရိတ်ကိုလျှော့ချလို့ရပါတယ်။

### 2. Preparing Environment And Caching
1. `runs-on: ubuntu-latest`: Job ကို Ubuntu latest environment ပေါ်မှာ run ပါမယ်။
2. `uses: actions/setup-go@v6`: Go v1.25.5 ကို install လုပ်ပါမယ်။
3. `actions/cache@v5`: `go.sum` file မှာ အပြောင်းအလဲမရှိသ၍ Go modules တွေကို ပြန်ပြီး download လုပ်စရာမလိုတော့ဘဲ cache ကနေ ပြန်ယူသုံးတဲ့အတွက် build time ကို သိသိသာသာ လျှော့ချပေးပါတယ်။ အချိန်ကုန်ငွေကုန်သက်သာစေတဲ့ အဓိကအချက်ပါ။

### 3. Code Formatting
```yaml
FILES=$(gofmt -s -l $(go list -f '{{.Dir}}' ./...))
if [ -n "$FILES" ]; then
  echo "❌ Formatting issues found:"
  echo "$FILES"
  exit 1
fi
echo "✅ Code formatting is correct"
```
1. `gofmt -s -l`: သေချာ format မလုပ်ထားတဲ့ file list ကို ထုတ်ပေးပါတယ်။
2. `if [ -n "$FILES" ]; then`: `$FILES` ထဲမှာ file တွေပါနေတယ်ဆိုရင် workflow ကို `exit 1` နဲ့ fail လိုက်မှာပါ။ ဒါက format မမှန်တဲ့ code တွေ `main` branch ထဲရောက်မသွားအောင် ကာကွယ်ထားတာပါ။

### 4. Static Code Analysis
`go vet ./...`: format string မှားတာ(`%s` နေရာမှာ `%d` လိုမျိုး format specifier မှားသုံးထားတာမျိုး)တွေ၊ shadowed variable တွေ၊ struct tag မှားသုံးထားတာတွေ၊ unreachable code တွေနဲ့ unsafe conversion တွေကို စစ်ပေးတာပါ။

### 5. Checking Unused Dependencies
```yaml
go mod tidy
if [ -n "$(git diff --name-only go.mod go.sum)" ]; then
  echo "❌ go.mod/go.sum are not tidy. Run 'go mod tidy' to fix:"
  git diff
  exit 1
fi
echo "✅ Dependencies are tidy"
```
1. `go mod tidy`: မလိုအပ်တဲ့ dependency တွေကို ရှင်းပေးပြီး လိုတာတွေကိုထည့်ပေးပါတယ်။
2. `if [ -n "$(git diff --name-only go.mod go.sum)" ]; then`: `git diff` နဲ့ `go.mod` ဒါမှမဟုတ် `go.sum` file တွေမှာ ပြောင်းလဲတာရှိမရှိ စစ်တာပါ။ ပြောင်းလဲတာရှိရင် (ဆိုလိုတာက developer က `go mod tidy` ကို မ run ခဲ့ဘူး) လို့ ယူဆပြီး workflow ကို `exit 1` နဲ့ fail လိုက်မှာပါ။

## နိဂုံး
ဒီ linting workflow က code quality နဲ့ issue ကို ၁၀၀% မစစ်ပေးနိုင်ပါဘူး။ PR review လုပ်တဲ့အခါမှာ အကူအညီဖြစ်အောင်နဲ့ reviewer မမြင်လိုက်တာတွေ ပါမသွားအောင် တဘက်တလမ်းကနေ ကူညီနိုင်တာပဲရှိတာပါ။ မှန်ကန်တဲ့ design pattern တွေသုံးဖို့၊ maintainability ကို ကျစေတဲ့ cognitive complexity နည်းဖို့တွေကတော့ ကိုယ်တိုင်ပဲလုပ်ရမှာပါ။ နောက်ပိုင်းကျရင် တခြားအသုံးဝင်တဲ့ workflow တွေလည်း ထပ်ရေးကြည့်ပါဦးမယ်။ အခုလက်ရှိ repo တွေမှာလည်း ကြည့်လို့ရပါတယ်။
