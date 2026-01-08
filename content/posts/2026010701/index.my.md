+++
title = "UUID Generator With AI Agent"
date = '2026-01-07T23:43:00+07:00'
draft = true
lang = "my"
slug = "uuid-generator-with-ai-agent"
tags = ["uuid", "react", "vite", "ai", "github-copilot"]
+++

## နိဒါန်း
ဒီရက်ပိုင်း React ဘက်ကိုပြန်စမ်းကြည့်ချင်တာနဲ့  sample project လေးတခုလုပ်လုပ်ဖို့စဉ်းစားမိပြီး [UUID Generator](https://github.com/pyaethu-aung/uuid-generator) website လေးတခုလုပ်ဖြစ်ခဲ့ပါတယ်။ GitHub Copilot Pro လည်း subscribe လုပ်ထားလက်စရှိတာနဲ့ GPT-5.1-Codex model သုံးပြီးပဲလုပ်ဖြစ်လိုက်တယ်။ အဓိကရည်ရွယ်ချက်ကတော့ GPT-5.1-Codex ကို စမ်းကြည့်ချင်တာရော top-down learning သုံးပြီး React ကို ပြန်လေ့လာချင်တာရောကြောင့်ပါ။

## အဓိက Features တွေ
1. **UUID Version Support**: V1 (timestamp-based)၊ V4 (random) နဲ့ V7 (time-ordered) ဆိုပြီး UUID version သုံးမျိုးကို support လုပ်ထားပါတယ်။
2. **Batch Generation**: တခါမှာ UUID အခု ၂၀၀ အထိ generate လုပ်နိုင်ပေမယ့် screen ပေါ်မှာတော့ အခု ၂၀ ပဲ ပြအောင် optimize လုပ်ထားပါတယ်။
3. **Copy & Download**: UUID တခုချင်းစီကို clipboard ဆီ copy လုပ်လို့ရသလို၊ batch တခုလုံးကိုလည်း text file အနေနဲ့ download လုပ်လို့ရပါတယ်။
4. **Formatting Options**: Uppercase လုပ်တာ၊ hyphens ဖယ်တာနဲ့ braces ထည့်တာတွေ လုပ်နိုင်ပါတယ်။
5. **Dark Mode**: System preference ပေါ်မူတည်ပြီး dark/light theme support ထည့်ထားပါတယ်။
6. **Responsive Design**: Mobile ကနေ desktop အထိ သုံးလို့ရအောင် responsive လုပ်ထားပါတယ်။

## Human Input
ဒီ project ကို စရေးကတည်းက အခြေခံလိုအပ်တာတွေကိုအရင်ထည့်ပြီးမှ ကျန်တာတွေကိုလိုက်ဖြည့်မယ်ဆိုပြီး စဉ်းစားခဲ့တာပါ။ ဒါပေမယ့် ပထမဆုံး prompy မှာကတည်းက လိုအပ်တာတွေကို တော်တော်လေးလုပ်သွားပေးနိုင်တယ်လို့ပြောရမှာပါ။ ကျွန်တော့အနေနဲ့ အောက်ကလေးချက်ပဲ အဓိကလုပ်ပေးခဲ့ရတယ်လို့ ပြောလို့ရပါတယ်။
1. **High-level requirements** တွေကို ရှင်းရှင်းလင်းလင်းရေးပေးတာ။
2. Generated code တွေကို review လုပ်ပေးတာ။
3. တဆင့်ပြီးတဆင့် **iterative prompt** တွေပေးပြီး refine လုပ်တာ။
4. Testing နဲ့  performance optimization အတွက် guidance ပေးတာ။

Copilot ကတော့ ဒါတွေကို အစအဆုံး implement လုပ်ပေးခဲ့ပါတယ်။
- React component architecture နဲ့ state management
- UI layout နဲ့ Tailwind styling
- UUID generation logic (`uuid` npm package)
- Clipboard API နဲ့ file download
- Theme switching
- Responsive design
- Unit tests (`vitest`)

## Prompts
သုံးခဲ့တဲ့ prompt တွေကို `README` မှာ ထည့်ပေးထားပါတယ်။
1. Build a Tailwind-forward UUID generator interface that feels polished and modern.
2. Add a selector that lets the user toggle between UUID versions v1, v4, and v7.
3. Keep badges/stat labels on a single line so that wording like "Characters Each" never wraps or overflows.
4. Provide animated feedback when copying UUIDs and acknowledge downloads with contextual text.
5. Extend download batches to 200 entries while keeping only 20 visible in the live preview.
6. Replace multiple controls with one slider that manages both preview and download counts, updating immediately as it moves.
7. Relocate version, batch size, and per-UUID character details into the stat cards above the list to avoid duplicate text elsewhere.

## Lessons Learned
ဒီ project ကနေ သင်ခန်းစာတွေလည်းရခဲ့ပါတယ်။

### 1. Clear Communication is Key
AI နဲ့အလုပ်လုပ်တဲ့အခါ prompt က ရှင်းရှင်းလင်းလင်းဖြစ်ဖို့ အရေးကြီးပါတယ်။ `Make it better` လို vague prompt တွေထက် `Keep badges on a single line to prevent wrapping` လို specific prompt တွေက ပိုကောင်းပါတယ်။

### 2. Iterative Development Works
တခါတည်း perfect solution ရအောင်မလုပ်ဘဲ၊ working version တခုကနေ တဖြည်းတဖြည်းချင်း improve လုပ်သွားတာက ပိုထိရောက်ပါတယ်။

### 3. Review and Validate
AI generated code တိုင်းကို စစ်ဖို့လိုပါတယ်။ Edge cases တွေ၊ performance issues တွေ၊ accessibility concerns တွေကို human developer က သတိထားကြည့်ရပါမယ်။

### 4. Documentation Matters
Development process တခုလုံးကို document လုပ်ထားတာက နောက်မှ ပြန်ကြည့်ရင် အသုံးဝင်ပါတယ်။

## နိဂုံး
အနာဂတ် development project တွေမှာ AI tools တွေကို ဘယ်လိုထိထိရောက်ရောက်သုံးမလဲဆိုတာ စဉ်းစားကြည့်သင့်ပါတယ်။ Productivity multiplier တခုအနေနဲ့ အလုပ်လုပ်နိုင်ပေမဲ့ developer ရဲ့ expertise နဲ့ judgment က အရေးကြီးနေဦးမှာပါ။

Live demo ကို [ဒီမှာ](https://pyaethu-aung.github.io/uuid-generator/) စမ်းကြည့်နိုင်ပြီး source code ကို [GitHub](https://github.com/pyaethu-aung/uuid-generator) မှာ လေ့လာနိုင်ပါတယ်။
