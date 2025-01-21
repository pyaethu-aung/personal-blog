+++
title = "Hot Reload in Go"
date = '2025-01-22T03:19:00+07:00'
draft = true
lang = "my"
slug = "hot-reload-in-go"
tags = ["development", "go"]
+++

Go လိုမျိုး compiled language အတော်များများမှာ hot reload က natively မပါတာများပါတယ်။ Flutter ကတော့ ချွင်းချက်ပေါ့။ အဲ့တော့ developer တွေအနေနဲ့က code ရေးလိုက် terminal မှာ ပြန်ပြီး recompile လုပ်လိုက်နဲ့က ပုံမှန်ဆို မသိသာပေမယ့် အကြိမ်ရေများလာတဲ့အခါကျ တော်တော်လေး စိတ်အနောင့်အယှက်ဖြစ်ရတယ်။ ကျွန်တော့လိုမျိုး code editor နဲ့ terminal နဲ့ကို သပ်သပ်စီသုံးတတ်တဲ့သူတွေကတော့ ပိုဆိုးမယ်ထင်ပါတယ်။ အဓိကပြဿနာတခုက context switching ပါ။ Complex ဖြစ်ပြီးတော့ nested iteration တွေပါတဲ့ code ကို develop/debug လုပ်နေရတာမျိုးမှာ အာရုံစိုက်ရခက်ပါတယ်။
