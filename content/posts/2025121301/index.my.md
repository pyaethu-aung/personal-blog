+++
title = "Go Linter GitHub Action"
date = '2025-12-13T22:59:00+07:00'
draft = true
lang = "my"
slug = "go-linter-github-action"
tags = ["github-action", "go"]
+++

နောက်ဆုံး blog post ရေးခဲ့တာက ပြီးခဲ့တဲ့ဇန်နဝါရီလကဆိုတော့ ဘာမှမရေးဖြစ်ခဲ့တာ တနှစ်နီးပါးလောက်တောင် ရှိသွားခဲ့ပါပြီ။ ဒီကြားထဲမှာ ကိုယ်ရေးကိုယ်တာကိစ္စတွေနဲ့ အာရုံများနေတာရော ပျင်းနေတာရောပါတယ်။ ဒီရက်ပိုင်းမှ [Bubble Tea](https://github.com/charmbracelet/bubbletea) ကို စမ်းကြည့်ချင်တာရော personal income tax တွက်ဖို့ CLI app လေးတခုရေးချင်တာရောနဲ့ [PIT calculator CLI app](https://github.com/pyaethu-aung/myanmar-pit-calculator လေးလုပ်ဖြစ်တယ်။ ရိုးရိုး CLI interface နဲ့ TUI interface ဆိုပြီး နှစ်ခုနဲ့ စမ်းကြည့်လို့ရပါတယ်။ Copilot အကူအညီနဲ့ ပါ README သေချာရေးဖြစ်သွားတော့ အဆင်ပြေရင် စမ်းကြည့်ကြစေချင်ပါတယ်။ Life insurance နဲ့ တခြား အခွန်သက်သာခွင့်တွေရယ် bonus နဲ့ တခြား taxable income တွေအတွက်ရယ်တော့ ထပ်ထည့်ဖို့တော့ ကျန်ပါဦးမယ်။

အခု post မှာတော့ PIT calculator မှာသုံးထားတဲ့ linter အတွက် GitHub action အကြောင်းကွက်ပြီး ပြောပြချင်လို့ပါ။ အလုပ်မှာကတော့ သုံးဖြစ်ပေမည့် personal project မှာတော့ Hugo ရဲ့ deployment အတွက်ကလွဲရင် အခုမှစသုံးဖြစ်တာပါ။