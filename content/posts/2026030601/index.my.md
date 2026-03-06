+++
title = "Past Commits တွေကို ဘယ်လို Update လုပ်မလဲ"
date = '2026-03-06T22:53:52+07:00'
draft = false
lang = "my"
slug = "update-past-commits"
tags = ["git", "github", "version-control"]
+++

## နိဒါန်း
Code push လုပ်တဲ့အခါ commit message တွေကို သေချာရေးဖို့ အရမ်းအရေးကြီးပါတယ်။ ဒါပေမယ့် တခါတလေမှာ commit message က typo ဖြစ်သွားတာမျိုး၊ ဒါမှမဟုတ် ပိုပြီးရှင်းလင်းတဲ့ commit message ကို ပြောင်းချင်တာမျိုးတွေ ကြုံရတတ်ပါတယ်။ 

ဒီ post မှာတော့ မိမိရေးခဲ့တဲ့ past commits တွေကို `git commit --amend` နဲ့ `git rebase` သုံးပြီး ဘယ်လို update လုပ်ရမလဲဆိုတာကို sharing လုပ်ချင်ပါတယ်။

## နောက်ဆုံး Commit ကို ပြင်ချင်တာဆိုရင်?
ကိုယ်ပြင်ချင်တဲ့ commit က latest commit ဖြစ်နေတယ်ဆိုရင်တော့ အလွယ်ဆုံးပါပဲ။

`git commit --amend` command ကိုသုံးပြီး အလွယ်တကူ ပြင်ဆင်နိုင်ပါတယ်။

```bash
git commit --amend
```

ဒီ command ကို run လိုက်တာနဲ့ terminal မှာ text editor ပွင့်လာပါလိမ့်မယ်။ အဲ့ဒီမှာ commit message အသစ်ကို ပြင်ရေးပြီး save လိုက်ရုံပါပဲ။

***Pro Tip:*** တကယ်လို့ commit message ကို မပြင်ဘဲ file တွေ မေ့ကျန်ခဲ့လို့ နောက်ဆုံး commit ထဲကိုပဲ ထပ်ထည့်ချင်တာဆိုရင်တော့ `git add .` လုပ်ပြီး `git commit --amend --no-edit` ဆိုပြီး အလွယ်တကူ သုံးလို့ရပါတယ်။

## နောက်ဆုံးမဟုတ်တဲ့ Commit အဟောင်းတွေကို ပြင်ချင်တာဆိုရင်?
တကယ်လို့ ကိုယ်ပြင်ချင်တဲ့ commit တွေက နောက်ဆုံး commit မဟုတ်ဘဲ အရင်ကလုပ်ခဲ့တဲ့ commit တွေဖြစ်နေတယ်ဆိုရင်တော့ `git rebase` ကို သုံးဖို့လိုပါတယ်။ ဥပမာ - နောက်ဆုံး commits ၃ ခုကို ပြန်ပြင်ချင်တယ်ဆိုရင် **interactive rebase** ကို အောက်ပါအတိုင်း သုံးနိုင်ပါတယ်။

```bash
git rebase -i HEAD~3
```

ဒီ command ကို run လိုက်တဲ့အခါ editor (maybe Vim) ပွင့်လာပြီး commit list ကို တွေ့ရပါလိမ့်မယ်။

ဥပမာ:
```text
pick e3a1b35 Commit message 1
pick 7ac9a67 Commit message 2
pick 4db8c21 Commit message 3
```

ကိုယ်ပြင်ချင်တဲ့ commit ရဲ့ရှေ့က `pick` ဆိုတဲ့စာသားနေရာမှာ `reword` လို့ ပြောင်းပေးရပါမယ်။ အခု ဥပမာမှာ `reword` ကို commit တိုင်းအတွက် သုံးပြထားပါတယ်။

```text
reword e3a1b35 Commit message 1
reword 7ac9a67 Commit message 2
reword 4db8c21 Commit message 3
```

Save လုပ်ပြီး ထွက်လိုက်တဲ့အခါ `reword` ပြောင်းထားတဲ့ commit တစ်ခုချင်းစီအတွက် editor က တလှည့်စီပွင့်လာပါလိမ့်မယ်။ အဲ့ဒီမှာ ကိုယ်ပြောင်းချင်တဲ့ commit message အသစ်တွေ လိုက်ပြင်ပြီး save လိုက်ရုံပါပဲ။

***Pro Tip:*** Interactive rebase မှာ `reword` အပြင် တခြားအသုံးဝင်တဲ့ command တွေလည်း ရှိပါသေးတယ်။ ဥပမာ:
- မလိုအပ်တဲ့ commit ကို ဖျက်ပစ်ချင်ရင် `drop` (သို့မဟုတ် `d`)
- commit နှစ်ခုကို တခုတည်းအဖြစ် ပေါင်းချင်ရင် `squash` (သို့မဟုတ် `s`)
- commit message တင်မဟုတ်ပဲ file တွေကိုပါ ပြင်ချင်တယ်ဆိုရင် `edit` (သို့မဟုတ် `e`)
- rebase လုပ်နေရင်း အမှားတစ်ခုခုဖြစ်သွားလို့ ဆက်မလုပ်တော့ဘူး၊ မူလအတိုင်း ပြန်ထားချင်တယ်ဆိုရင် `git rebase --abort` ကို သုံးနိုင်ပါတယ်။

## Safety Net
Commit history တွေကို rebase လုပ်ရင်း မှားသွားလို့ပဲဖြစ်ဖြစ်၊ မှားဖျက်မိသွားလို့ပဲဖြစ်ဖြစ် မူလအတိုင်း ပြန်လိုချင်တယ်ဆိုရင်လည်း စိတ်မပူပါနဲ့။ `git reflog` ဆိုတဲ့ command ရှိပါတယ်။

```bash
git reflog
```

ဒီ command ကတဆင့် ကိုယ်လုပ်ခဲ့သမျှ action history တွေနဲ့ commit hash တွေကို ပြန်ကြည့်လို့ရပါတယ်။ အဲ့ဒီကနေ ကိုယ်ပြန်သွားချင်တဲ့ နေရာကို `git reset --hard <commit-hash>` သုံးပြီး အလွယ်တကူ ပြန်သွားနိုင်ပါတယ်။

## Push to Remote
Commit တွေ ပြင်ပြီးသွားပြီဆိုရင်တော့ remote repository ကို ပြန် push လုပ်ဖို့ လိုပါတယ်။ ဒါပေမယ့် commit history တွေကို ပြောင်းလိုက်တဲ့အတွက် ရိုးရိုး `git push` လုပ်လို့ မရတော့ပါဘူး။ 

ဒီလိုအခြေအနေမျိုးမှာ **force push** လုပ်ဖို့လိုပေမယ့် တခြားသူတွေရဲ့ code တွေကို မထိခိုက်စေဖို့အတွက် ပို safe ဖြစ်တဲ့ `--force-with-lease` ကို အမြဲသုံးသင့်ပါတယ်။

```bash
git push --force-with-lease
```

## နိဂုံး
ဒီ command တွေက နေ့စဉ် development workflow မှာ အရမ်းအသုံးဝင်ပါတယ်။ မှားရေးမိတဲ့ commit တွေကို code review မလုပ်ခင် အလွယ်တကူ ပြင်ဆင်နိုင်တာကြောင့် git history ကိုလည်း clean and clear ဖြစ်စေမှာပါ။
