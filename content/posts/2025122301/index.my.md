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
