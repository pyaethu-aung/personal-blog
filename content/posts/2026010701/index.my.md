+++
title = "UUID Generator With AI Agent"
date = '2026-01-07T23:43:00+07:00'
draft = false
lang = "my"
slug = "uuid-generator-with-ai-agent"
tags = ["uuid", "react", "vite", "ai", "github-copilot"]
+++

## နိဒါန်း
AI assistant တွေ ပေါ်လာတာနဲ့အမျှ software development landscape က တော်တော်လေးပြောင်းလဲလာပါတယ်။ အထူးသဖြင့် GitHub Copilot လို tool တွေက simple CRUD app ကနေ complex logic ပါတာတွေအထိ implementation အပိုင်းကို handle လုပ်ပေးနိုင်နေပါပြီ။ Developer တွေအနေနဲ့ high-level requirements တွေကိုပဲ focus လုပ်ဖို့ လိုပါတော့တယ်။

ဒီ post မှာတော့ GitHub Copilot ကိုသုံးပြီးရေးထားတဲ့ [UUID Generator](https://github.com/pyaethu-aung/uuid-generator) project အကြောင်း sharing လုပ်ချင်ပါတယ်။

## UUID Generator Features
[UUID Generator](https://pyaethu-aung.github.io/uuid-generator/) ကို React 19, Vite နဲ့ Tailwind CSS သုံးပြီး build လုပ်ထားပါတယ်။ အဓိက feature တွေကတော့-
1. **UUID Version Support**: V1 (timestamp-based)၊ V4 (random) နဲ့ V7 (time-ordered) ကို support ပေးပါတယ်။
2. **Batch Generation**: တခါ generate လုပ်ရင် UUID အခု 200 အထိ ရနိုင်ပါတယ်။ UI ပေါ်မှာတော့ performance အတွက် 20 ခုပဲ preview ပြပေးပါတယ်။
3. **Smart Controls**: Batch size နဲ့ preview count ကို unified slider နဲ့ control လုပ်နိုင်ပါတယ်။
4. **Copy & Download**: Clipboard copy နဲ့ text file download (timestamped filename နဲ့) ရပါတယ်။
5. **Formatting**: Uppercase, remove hyphens, wrap braces စတာတွေ support လုပ်ပါတယ်။

## Breaking Down
### 1. AI-First Development
ဒီ project ရဲ့ ထူးခြားချက်က core implementation အားလုံးနီးပါးကို **GitHub Copilot** (GPT-5.1-Codex) က ရေးပေးထားတာပါ။ Human inputs အနေနဲ့က high-level requirements, code review နဲ့ testing guidance လောက်ပဲ ပေးရပါတယ်။ React architecture, Tailwind styling, Business logic နဲ့ Unit tests တွေကို AI က အစအဆုံး implement လုပ်သွားပါတယ်။

### 2. Prompt Evolution
Workflow က traditional planning -> coding ပုံစံမဟုတ်ဘဲ **conversational iteration** ပုံစံနဲ့သွားပါတယ်။ README မှာလည်း document လုပ်ထားတဲ့ prompt history ကိုကြည့်ရင် feature တခုချင်းစီကို incremental build လုပ်သွားတာ တွေ့ရပါလိမ့်မယ်။
1. Initial Interface styling
2. Version selector logic
3. Mobile responsive badges
4. UX interactions (copy feedback)
5. Scaling to 200 items
6. Unified slider control
7. Information architecture refinement

### 3. Technical Highlights
Copilot ရေးပေးတဲ့ code quality ကလည်း မဆိုးဘူးလို့ ပြောလို့ရပါတယ်။
- **Custom Hooks**: logic တွေကို `useTheme` နဲ့ `useUuidGenerator` ဆိုပြီး ခွဲထုတ်ထားတဲ့အတွက် clean code ဖြစ်သလို test လုပ်ရတာလည်း လွယ်ကူပါတယ်။
- **Web Crypto API**: True randomness ရဖို့ `Math.random()` အစား `crypto.randomUUID()` ကို web crypto API ကနေတဆင့် သုံးထားပါတယ်။
- **Optimized Rendering**: item 200 လုံးကို render မလုပ်ဘဲ viewport ထဲက item တွေကိုပဲပြပေးတဲ့ logic တွေပါ ထည့်သွင်းစဉ်းစားပေးထားပါတယ်။

## နိဂုံး
ဒီ project က AI-assisted development workflow ကို စမ်းသပ်ဖို့ ကောင်းတဲ့ example ပါ။ Developer အနေနဲ့ implementation details ထက် architectural vision နဲ့ quality assurance ပေါ် ပို focus လုပ်နိုင်တာကို တွေ့ရပါတယ်။ Project source code ကို [GitHub](https://github.com/pyaethu-aung/uuid-generator) မှာ လေ့လာနိုင်ပြီး [Demo](https://pyaethu-aung.github.io/uuid-generator/) ကိုလည်း ဝင်ရောက်စမ်းသပ်ကြည့်နိုင်ပါတယ်။
