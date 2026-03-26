+++
title = "Claude Code အသုံးပြုသူများအတွက် အသုံးဝင်မယ့် claude-replay အကြောင်း"
date = '2026-03-24T23:00:00+07:00'
draft = false
lang = "my"
slug = "claude-replay-for-claude-code"
tags = ["claude", "claude code", "claude-replay", "tools"]
+++

[Read this article in English]({{< relref path="2026032401/index.en.md" lang="en" >}})

## နိဒါန်း

အခုလက်ရှိက Claude Code ရော Antigravity ရောကို အလုပ်အတွက်ရယ် personal အတွက်ရယ်နဲ့ သုံးဖြစ်နေတယ်။ အဲ့မှာ တခုပြဿနာရှိလာတာက session ထဲက conversation တွေကို တခြားတယောက်ကို share ချင်တာမျိုးဆိုရင် အစဉ်မပြေတာပါပဲ။ Log အရှည်ကြီးတွေ ပို့ရတာ ဒါမှမဟုတ် screen record ဖမ်းပြီး ပို့ရတာမျိုးက သိပ်အဆင်မပြေပါဘူး။

အဲဒါနဲ့ ရှာရင်းဖွေရင်း [claude-replay](https://github.com/es617/claude-replay) ဆိုတဲ့ tool တခုသွားတွေ့တယ်။ **claude-replay** က AI နဲ့ chat session တွေကို video လိုကြည့်လို့ရအောင် interactive playback HTML file ထုတ်ပေးတာပါ။

လက်ရှိတော့ Claude Code၊ Cursor၊ နဲ့ Codex CLI တွေကိုပဲ လုပ်ပါသေးတယ်။

## အသုံးဝင်တဲ့ command တွေ
အသုံးဝင်တဲ့ command တွေအနေနဲ့ဆိုရင်:
- `claude-replay`: မှာ လက်ရှိ တွေကို ကြည့်လို့ရမယ်
- `claude-replay abc123def456 -o replay.html`: ပေးပြီး Interactive playback HTML file ထုတ်လို့ရမယ်
- `claude-replay session.jsonl --no-thinking --no-tool-calls -o replay.html`: တွေ မလိုချင်ရင် ဖြုတ်ခဲ့လို့ရမယ်
- `claude-replay session.jsonl --turns 5-15 --speed 2.0 -o replay.html`: လိုချင်တဲ့ အပိုင်းလေးကိုပဲ အမျိုးမျိုးနဲ့ ပြလို့ရမယ်
- `claude-replay session.jsonl --theme dracula -o replay.html`: လည်း ပြောင်းလို့ရပါမယ်

## နိဂုံး
[Secret redaction](https://github.com/es617/claude-replay?tab=readme-ov-file#secret-redaction) နဲ့ တခြားအသုံးဝင်တာတွေရယ် single HTML file ဖြစ်တဲ့အတွက် တခြား page တခုခုမှာ embed လုပ်တာတွေရယ်ကို [claude-replay](https://github.com/es617/claude-replay) GitHub Repo မှာကြည့်ပြီး စမ်းကြည့်စေချင်ပါတယ်။