+++
title = "GitHub မှာ Contribution တွေ မပေါ်ရင် ဘယ်လိုလုပ်မလဲ (gitmailmap)"
date = '2026-03-09T22:00:40+07:00'
draft = false
lang = "my"
slug = "github-contributions-missing-gitmailmap"
tags = ["git", "github", "gitmailmap", "filter-repo"]
+++

## နိဒါန်း

GitHub သုံးတဲ့သူတိုင်း ကိုယ့်ရဲ့ profile မှာ စိမ်းနေတဲ့ contribution graph လေးတွေကို မြင်ချင်မှာပါ။ ဒါပေမယ့် တခါတလေမှာ ကိုယ်က နေ့တိုင်း code တွေ ရေးပြီး push လုပ်နေပေမယ့် GitHub contribution မှာ သွားမပေါ်တာမျိုး ကြုံရနိုင်ပါတယ်။ အဓိကအကြောင်းအရင်းကတော့ ကိုယ့်ရဲ့ local Git config မှာ သုံးထားတဲ့ email နဲ့ GitHub အကောင့်မှာရှိတဲ့ email မတူလို့ပါ။

ဒီ post မှာတော့ ဘာကြောင့်အဲ့လိုဖြစ်ရလဲဆိုတာကို ပြန်ကြည့်ပြီး [`gitmailmap`](https://git-scm.com/docs/gitmailmap) နဲ့ `git filter-repo` command တွေ သုံးပြီး အရင်ကလုပ်ခဲ့တဲ့ past commits တွေကိုပါ ဘယ်လိုပြန်ပြင်မလဲဆိုတာကို sharing လုပ်ချင်ပါတယ်။

## Commit တွေ ဘာလို့မပေါ်တာလဲ?
### 1. Local နဲ့ GitHub ပေါ်က Email တွေကို စစ်ပါ
ပထမဆုံးအနေနဲ့ ကိုယ့်ရဲ့ commit တွေ ဘာလို့ မပေါ်လဲဆိုတာကို စစ်ကြည့်ရအောင်။

Terminal မှာ အောက်က command ကို run ပြီး လက်ရှိ local git မှာ setup လုပ်ထားတဲ့ email ကို စစ်ကြည့်ပါ။

```bash
git config user.email
```

အဲ့ဒီမှာ ပေါ်လာတဲ့ email က GitHub ပေါ်က Settings -> Emails ထဲမှာ ထည့်ထားတဲ့ email နဲ့ တူရပါမယ်။ အဲ့ email က “Verified” လည်း ဖြစ်နေဖို့ လိုပါတယ်။

### 2. Commits တွေ GitHub ပေါ်ရောက်မရောက် စစ်ပါ

Local မှာပဲ commit လုပ်ပြီး `git push` မလုပ်ရသေးဘူးဆိုရင်လည်း contribution မှာ မပေါ်ပါဘူး။ နောက်ပြီး မလိုအပ်တဲ့ branch တွေမှာ commit လုပ်ထားပြီး default branch (များသောအားဖြင့် `main` သို့မဟုတ် `master`) ကို merge မလုပ်ရသေးရင်လည်း မပေါ်တတ်ပါဘူး။ ပြီးတော့ private rep တွေမှာ contribution တွေကို profile မှာ မပေါ်အောင် ပိတ်ထားတာမျိုးလည်း ဖြစ်နိုင်ပါတယ်။

## Email တွေ မတူခဲ့ဘူးဆိုရင်?
ပြဿနာက email မတူတာဆိုတာ သေချာသွားပြီဆိုရင်တော့ နောက်ပိုင်းထပ်လာမဲ့ commit တွေအတွက် email အမှန်ကို အရင်ပြောင်းပေးရပါမယ်။

```bash
git config --global user.name "Your Name"
git config --global user.email "your-real-email@example.com"
```

ဒီလိုပြောင်းလိုက်တာနဲ့ နောက်ထပ် အသစ်လုပ်မယ့် commit တွေအားလုံးက GitHub မှာ အလိုအလျောက် contribution အနေနဲ့ ပေါ်လာပါလိမ့်မယ်။ လက်ရှိ project တခုတည်းအတွက်ပဲ သီးသန့်ထားချင်ရင်တော့ `--global` မပါဘဲ သုံးပေးပါ။

## အရင်ကလုပ်ခဲ့တဲ့ Commits တွေကို ဘယ်လိုပြင်မလဲ?
ပြဿနာက အရင်က email မပြောင်းထားခင်က လုပ်ခဲ့တဲ့ commit အဟောင်းတွေပါ။ သူတို့ထဲမှာ email အမှားတွေက ပါသွားပြီးပါပြီ။ အဲ့ဒါတွေကို date-time တွေ မပြောင်းစေပဲ email နဲ့ name အမှန်ပြန်ပြောင်းချင်တယ်ဆိုရင် `.mailmap` နဲ့ `git filter-repo` ကို တွဲသုံးဖို့ လိုပါတယ်။

### 1. `git filter-repo` ကို Install လုပ်ပါ

`git filter-repo` က Git ရဲ့ built-in filter-branch ထက် ပိုမြန်ပြီး ပို safe ဖြစ်တဲ့ tool တခုပါ။

- macOS မှာဆိုရင် `brew install git-filter-repo`
- Linux သို့မဟုတ် Windows မှာဆိုရင် `pip install git-filter-repo` နဲ့ install လုပ်နိုင်ပါတယ်။

*ဘာမှမလုပ်ခင် ကိုယ့်ရဲ့ repository ကို local မှာ backup ယူထားဖို့ ပေးချင်ပါတယ်။*

### 2. .mailmap File လုပ်ပါ

ကိုယ့် project ရဲ့ root directory မှာ `.mailmap` ဆိုတဲ့ file တခုဆောက်ပါ။ အဲ့ဒီ file ထဲမှာ အောက်က format အတိုင်း ထည့်ပေးပါ။

```text
Kyaw Kyaw Myo <your-real-email@example.com> <user@github.com>
```

ဒီနေရာမှာ:
- `Kyaw Kyaw Myo` နေရာမှာ မကိုယ့်ရဲ့ နာမည်အမှန်
- `your-real-email@example.com` နေရာမှာ GitHub မှာ verified ဖြစ်နေတဲ့ email အမှန်
- `user@github.com` နေရာမှာ အရင်က သုံးခဲ့မိတဲ့ မှားနေတဲ့ email ကို ထည့်ပါ။

### 3. History ကို Rewrite လုပ်ပါ

`.mailmap` file ကို သိမ်းပြီးပြီဆိုရင် အောက်က command ကို run ပါ။

```bash
git filter-repo --mailmap .mailmap
```

ဒီ command က commit history တွေကို လိုက်ဖတ်ပြီး `.mailmap` ပါ အချက်အလက်တွေအတိုင်း အရင်က မှားခဲ့တဲ့ email တွေကို အမှန်ပြန်ပြောင်းပေးသွားမှာပါ။ မူလ commit date တွေလည်း ပြောင်းသွားမှာ မဟုတ်ပါဘူး။

## Force Push
Commit history တွေကို rewrite လုပ်လိုက်တာဖြစ်တဲ့အတွက် ရိုးရိုး `git push` နဲ့ တင်လို့ မရတော့ပါဘူး။ Force push လုပ်ပေးဖို့ လိုပါတယ်။

```bash
git push --force-with-lease --all origin
git push --force-with-lease --tags origin
```

*ဒါဟာ repo ရဲ့ history ကို rewrite လိုက်တာမလို့ ကိုယ်တယောက်တည်း သုံးတဲ့ personal repo တွေမှာပဲ လုပ်သင့်ပါတယ်။ တကယ်လို့ အဖွဲ့လိုက်လုပ်နေတဲ့ project ဆိုရင်တော့ project ကို clone ထားတဲ့ တခြား team member တွေပါ ရိုက်ခတ်မှုရှိနိုင်လို့ သတိထားပါ။*

## နိဂုံး
Force push လုပ်ပြီး ခဏလောက်စောင့်လိုက်တာနဲ့ မိမိရဲ့ GitHub profile ကို သွားကြည့်ရင် အရင်က ပျောက်နေတဲ့ green block လေးတွေ ပြန်ပေါ်လာတာကို တွေ့ရပါလိမ့်မယ်။ ဒီ post က contribution graph မှာ contribution တွေ မပေါ်လို့ စိတ်ညစ်နေရတဲ့ developer တွေအတွက် အထောက်အကူဖြစ်မယ်လို့ မျှော်လင့်ပါတယ်။
