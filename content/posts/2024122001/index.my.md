+++
title = "Regular Expressions အခြေခံ"
date = '2024-12-29T02:41:08+07:00'
draft = false
lang = "my"
slug = "regular-expressions-101"
tags = ["regex", "python"]
+++

[Read this article in English]({{< relref path="2024122001/index.en.md" lang="en" >}})

[အပိုင်း ၂]({{< relref path="2025010501/index.my.md">}})

## Introduction
`([A-Za-z])\w+\s+\W\D`  
အပေါ်ကလိုမျိုး regular expression ကိုတွေ့ရင် programmer အတော်များများက ဘာအဓိပ္ပါယ်လဲလို့ မသိကြတာများမယ်ထင်တယ်။ ကျွန်တော်လည်းပါတယ်။ ပြီးတော့ validation စစ်ဖို့အတွက် regular expression ရေးရတော့မယ်ဆိုရင် [RegExr](https://regexr.com/) တို့ [regex101](https://regex101.com/) တို့မှာ သွားရေးကြတာ များမလားပဲ။ အခုဆိုရင်တော့ Chat GPT တို့ GitHub Copilot တို့ပေါ့။

Data နဲ့ပတ်သက်တဲ့ course တခု တက်ဖြစ်ရင်းနဲ့  regular expression အကြောင်းပါလာတော့ ကိုယ်တိုင်လည်းမှတ်မိအောင်ဆိုပြီး ဒီ post ကို ရေးထားလိုက်တာပါ။ ဒီ post မှာက Python သုံးပြီး ရေးထားပေမဲ့ regular expression နဲ့ ပတ်သက်တာတွေက တခြား language တွေမှာလည်း အတူတူပါပဲ။

## General Tokens
အခြေခံအနေနဲ့ ဒီ token တွေသိထားရင် pattern တော်တော်များများကို match လုပ်လို့ရတယ်။ တခြား token တွေလည်း ရှိသေးပေမဲ့ အသုံးနည်းလို့ ချန်ထားခဲ့ပါမယ်။
- `\d`: **0** to **9** အထိ ဘယ် digit မဆိုအတွက်ပါ။
- `\s`: **space**၊ **tab** နဲ့ **new line** character တွေအတွက်ပါ။
- `\w`: ဒါက word character အတွက်ပါ။ **a** to **z**၊ **A** to **Z**၊ **0** to **9** နဲ့ **_** character တွေပါပါမယ်။
- `.`: **new line** ကလွဲရင် ကျန်တဲ့ တွေ အားလုံးပါပါမယ်။

Token တွေက small letter တွေပဲဖြစ်နေတာကို သတိထားမိမယ်ထင်တယ်။ Capital letter ပြောင်းလိုက်မယ်ဆိုရင် ပြောင်းပြန်ဖြစ်သွားမှာပါ။
- `\D`: **0** to **9** အထိ digit ကလွဲရင် ကျန်တဲ့ ဘယ် character မဆိုအတွက်ပါ။
- `\S`: **space**၊ **tab** နဲ့ **new line** ကလွဲရင် ကျန်တဲ့ ဘယ် character မဆိုအတွက်ပါ။
- `\W`: **a** to **z**၊ **A** to **Z**၊ **0** to **9** နဲ့ **_** ကလွဲရင် ကျန်တဲ့ ဘယ် character မဆိုအတွက်ပါ။

အောက်က Python code ကို run ကြည့်ရင် result က `['H', 'i', '1', '2', '3']` ဆိုပြီးရပါမယ်။ `re.findall()` function ရဲ့ ပထမ parameter က regular expression ပါ။
```python
import re

print(re.findall(r'\w', 'Hi 123'))
```
Regular expression ကို `r'\w\w'` လို့ ပြောင်းကြည့်ရင်တော့ `['Hi', '12']` ဆိုပြီး ရမယ်။ Word character နှစ်ခု ဆက်တိုက်ဖြစ်ရမယ်လို့ match လုပ်လိုက်တာမလို့ပါ။ `r'\w\s\d'` လို့ ပြောင်းကြည့်ရင် `['i 1']` တခုပဲ ရပါမယ်။

ဒီနေရာမှာ တခုမေးဖို့ရှိတာက ဖုန်းနံပါတ်လိုမျိုးမှာ digit ကိုးခု ရှိရမယ်လို့ match လုပ်ချင်ရင် `\d` ကို ကိုးခါဆက်တိုက် ရေးရမှာလား၊ character သုံးခုကနေ ငါးခုရှိတာကို match လုပ်ချင်တယ်ဆိုရင်ရော ဘယ်လိုလုပ်ရမှာလဲလို့ စဉ်းစားနိုင်တယ်။ အဲဒီလိုအခြေအနေမျိုးအတွက်က **quantifier** ကို တွဲပြီးသုံးရမှာပါ။

## General Quantifiers
အောက်က quantifier တွေက အရိုးရှင်းဆုံးနဲ့ အများဆုံးသုံးကြတာတွေပါ။
- `?`: **Zero** or **one** occurrence.
- `*`: **Zero** or **more** occurrences.
- `+`: **One** or **more** occurrences.
- `{5}`: Exactly **five** occurrences.
- `{5,9}`: Between **five** and **nine** occurrences.
- `{5,}`: **five** or **more** occurrences.

```python
import re

print(re.findall(r'\d+', 'Hi 123 4567 89101112'))
```
ဒီ Python example မှာဆိုရင် digit တခု ဒါမှမဟုတ် တခုထက်ပိုပြီး ဆက်တိုက်လာဖို့ကို match လုပ်လိုက်တာမလို့ ရလာဒ်က `['123', '4567', '89101112']` ဖြစ်မှာပါ။

တကယ်လို့ `r'\d{4}'` နဲ့ match လုပ်ရင်တော့ ရလာဒ်က `['4567', '8910', '1112']` ဖြစ်ပြီး `r'\d{4,}'` နဲ့ match လုပ်ရင်တော့ `['4567', '89101112']` ဖြစ်လိမ့်မယ်။

ဒီမှာ တခုသတိထားသင့်တာက quantifier တွေက **immediate to the left** ပါ။ Quantifier ရဲ့ ဘယ်ဘက်ကပ်လျက်မှာရှိနေတဲ့ character ဒါမှမဟုတ် token ပေါ်မှာပဲ သက်ရောက်ပါလိမ့်မယ်။
```python
import re

print(re.findall(r'1\w+', '1a1b1c1d1e1f1g'))
```
ဒီ Python code မှာဆိုရင် ရလာဒ်က  `['1a', '1b', '1c', '1d', '1e', '1f', '1g']` မဟုတ်ပဲ `['1a1b1c1d1e1f1g']` ဖြစ်မှာပါ။ `1` နဲ့  `\w` နှစ်ခုတွဲပေါ်မှာ `+` quantifier က သက်ရောက်တာမဟုတ်ပဲ `1` အနောက်မှာ တခု ဒါမှမဟုတ် တခုထက်ပိုတဲ့ character တွေလိုက်ရမယ်လို့ match လုပ်ထားတာမလို့ပါ။

## Excaping Special Character
```python
import re

print(re.findall(r'.\s', 'This is first sentence. And this is second sentence.'))
```
ဒီ Python code က အမှန်တော့ **full stop** အနောက်မှာ့ **space** ကပ်ပါလားဆိုတာကို စစ်ချင်တာ။ ဒါပေမဲ့ `.` ကို any character လို့ ယူဆသွားတဲ့အတွက် ရလာဒ်က `['s ', 's ', 't ', '. ', 'd ', 's ', 's ', 'd ']`  ဖြစ်နေပါလိမ့်မယ်။ Special character ဖြစ်တဲ့ `.` ကို escape လုပ်ဖို့ `\` ကိုသုံးပြီး `r'\.\s'` လို့ပြင်လိုက်ရင်တော့ `['. ']` ရလာဒ်ရလာမှာပါ။

## Last But Not Least
နောက်ထပ်အသုံးဝင်တဲ့ token နှစ်ခုက `^` နဲ့ `$` ပါ။
- `^`: Matches the start of the string.
- `$`: Matches the end of the string.

```python
import re

print(re.findall(r'hello_\d+', 'hello_world hello_123'))
```
ဒီ Python code မှာဆိုရင် ရလာဒ်က `['hello_123']` ဖြစ်လိမ့်မယ်။ တကယ်လို့ `hello_'` နဲ့ အစပြုထားပြီး အနောက်ကနေ တခုထက်ပိုတဲ့ digit လိုက်လားဆိုတာမျိုးကို စစ်ချင်တာဆိုရင် `r'^hello_\d+'` ကို သုံးမှမှန်ပါလိမ့်မယ်။

အဲဒီလိုပဲ `hello_'` အနောက်ကနေ တခုထက်ပိုတဲ့ digit နဲ့ အဆုံးသတ်ထားလား စစ်ချင်ရင်တော့ `r'hello_\d+$'` ကို သုံးရမှာပါ။

## Python Functions
ဒီ post က regular expression ကိုပဲ အဓိကပြောချင်တာဖြစ်လို့ `re.findall()` function ကိုပဲသုံးထားလိုက်တာပါ။ အောက်က Python function တွေကိုလဲ လိုအပ်ချက်ပေါ်မူတည်ပြီး စမ်းသုံးကြည့်ပါ။
- `re.match()`: Checks for a match only at the start of the string.
- `re.search()`: Searches the entire string for the first match.
- `re.sub()`: Replaces occurrences of a pattern with a specified string.
- `re.split()`: Splits the string at each match of the pattern.

## Outroduction
**Regular Expression အခြေခံ** ဆိုတဲ့အတိုင်း ဒီမှာရေးခဲ့တာတွေကို ကျွမ်းကျွမ်းကျင်ကျင်သုံးနိုင်ဖို့က များများလေ့ကျင့်ဖို့လိုပါတယ်။ Email နဲ့ ID number လိုမျိုးတွေကို များများစမ်းပြီး match လုပ်ကြည့်ပါ။ နောက် post တွေကျရင် password validation လိုမျိုး ပိုပြီးရှုပ်ထွေးတဲ့ pattern တွေကို match လုပ်နိုင်ဖို့အတွက် ရေးပါဦးမယ်။

## References:
- [Regular Expression in Python by Data Camp](https://campus.datacamp.com/courses/regular-expressions-in-python)
- [regex101.com](https://regex101.com/)
