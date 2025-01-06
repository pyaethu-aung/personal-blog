+++
title = "Regular Expressions အခြေခံ (အပိုင်း ၂)"
date = '2025-01-05T02:38:00+07:00'
draft = true
lang = "my"
slug = "regular-expressions-101-part-02"
tags = ["regex", "python"]
+++

[အပိုင်း ၁]({{< relref path="2024122001/index.my.md" lang="my" >}})

## Pipe Metacharacter
အပိုင်း ၁ မှာပြောခဲ့တဲ့ `.`၊ `?`၊ `+`၊ `*`၊ `^` နဲ့ `$` metacharacter တွေအပြင် အခြား အသုံးဝင်တာတခုက `|` ပါ။
```python
import re

print(re.findall(r'Go|Python', 'I\'m intrested in Go, JavaScript, Python, and SQL'))
```
အခြား programming language တွေမှာလိုပဲ `|` ကို **OR operator** အနေနဲ့ မြင်ကြည့်လို့ရပါတယ်။ အပေါ်က Python code မှာဆိုရင် **Go** ဒါမှမဟုတ် **Python** လို့ match လုပ်လိုက်တာဆိုတော့ result က `['Go', 'Python']` ဖြစ်မှာပါ။
