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

## Character Classes
### Set
Square bracket `[]` ကို regular expression မှာ **set** နဲ့ **range** အတွက်သုံးပါတယ်။
```python
import re

print(re.findall(r'[nl]ot', 'Not not Hot hot Lot lot'))
```
ဒီ code မှာဆိုရင် **n** ဒါမှမဟုတ် **l** နဲ့အစပြုထားရမယ်၊ အနောက်ကနေ **ot** ကပ်လိုက်ရမယ်လို့ match လုပ်ထားလိုက်တာပါ။ Result အနေနဲ့ `['not', 'lot']` လို့ ရပါလိမ့်မယ်။

`[nl]` expression ကို case insensitive ဖြစ်စေချင်တယ်ဆိုရင် အပေါ်က [pipe character](#pipe-metacharacter) ကိုသုံးပြီး ဒီလိုရေးလို့ရပါတယ်။
```python
import re

print(re.findall(r'[N|nL|l]ot', 'Not not Hot hot Lot lot'))
```
Result အနေနဲ့ `['Not', 'not', 'Lot', 'lot']` ဆိုပြီးရလာပါလိမ့်မယ်။ ဒီနေရာမှာ `|` မသုံးပဲနဲ့ case **ignorecase flag** သုံးပြီးလည်း ရေးလို့ရပါတယ်။ Python မှာကတော့ အောက်ကလို parameter အနေနဲ့ ပေးရပါတယ်။
```python
import re

print(re.findall(r'[nl]ot', 'Not not Hot hot Lot lot', re.IGNORECASE))
```

### Range
```python
import re

if re.fullmatch(r'[A-Za-z0-9]+', 'NoSpaceAndSpecialCharacter0123456789'):
    print('Match')
else:
    print('Not match')
```
ဒီလိုမျိုး **alphabat** နဲ့ **number** တွေပဲပါရမယ်၊ တခြား **space** နဲ့ **special character** တွေမပါရဘူးဆိုတဲ့ regular expression မျိုးကို programmer တော်တော်များများ သုံးဖူးကြမှာပါ။ ဒီမှာဆိုရင် `print('Match')` ကို run သွားပါလိမ့်မယ်။ တကယ်လို့ **space** နဲ့ **special character** တွေ ပါလာမယ်ဆိုရင်တော့ `print('Not match')` ကို run သွားမှာပါ။

### Not In `^`
`^` ကို **start of the string** အနေနဲ့ [အပိုင်း ၁]({{< relref path="2024122001/index.my.md" lang="my" >}}) မှာ ပြောခဲ့ပါတယ်။ သူ့ကို set နဲ့ range မှာ **not in** အနေနဲ့လည်း သုံးပါတယ်။
```python
import re

print(re.findall(r'[^A-Za-z0-9]+', 'NoSpaceAndSpecialCharacter#!0-0 123456789'))
print(re.findall(r'[^nl]ot', 'not hot lot'))k
```
ဒီ code မှာဆိုရင် ပထမ print မှာ **alphabatic** ရော **digit** ရောမဟုတ်ရဘူးဆိုတဲ့အတွက် result က `['#!', '-', ' ']` ရပါလိမ့်မယ်။ ဒုတိယ print မှာတော့ **n** နဲ့ရော **l** နဲ့ရော မစရတဲ့အတွက် result က `['hot']` ဖြစ်မှာပါ။

## Greedy and Non-Greedy Quantifiers
### Greedy Quantifier
Standard quantifier တွေဖြစ်တဲ့ `.`၊ `?`၊ `+`၊ `*` နဲ့ `{from, to|` အစရှိသဖြင့်တွေက default အနေနဲ့ greedy ပါ။ Greedy ဆိုတာက match ဖြစ်နေသေးသ၍ ရနိုင်သလောက် match လုပ်တာကို ပြောတာပါ။ အခုပြောခဲ့တာက ရှုပ်နေသလိုဖြစ်ပေမဲ့ အောက်က code ကိုကြည့်လိုက်ရင်ရှင်းပါလိမ့်မယ်။
```python
import re

print(re.findall(r'\w+', 'abcdefh123!@#'))
```
ဒီ code ကို run ကြည့်လိုက်ရင် result က သိထားပြီးဖြစ်တဲ့အတိုင်း `['abcdefh123']` လို့ထွက်မှာပါ။ `r'\d+'` လို့ပြောင်းပြီး match လိုက်ရင်တော့ result က `['123']` ဖြစ်မှာပါ။

ဒီနေရာမှာ `+` quantifier က **a** ကနေစပြီး match လို့ရသလောက် match သွားတာပြီး `!` ကျမှ match မဖြစ်တော့တဲ့အတွက် ဒီ result ရလာတယ်ထင်ရင် အပြည့်မမှန်ပါဘူး။ ဘာဖြစ်လို့လဲဆိုတာ နောက်ဥပမာတခုနဲ့ ဆက်ပြောပြပါမယ်။
```python
import re

print(re.findall(r'.*hello', 'xhello123'))
```
ဒီ code မှာဆိုရင် `.*` က sentence တခုလုံးကို match ဖြစ်နေပြီးတော့ result က `['xhello123']` ရမယ်လို့ ထင်ရမှာပါ။ ဒါပေမဲ့ `.*` မှာ sentence တခုလုံး match ဖြစ်နေပြီး ကျန်တဲ့ token တွေကို backtrack နဲ့ တခုစီ process လုပ်သွားတာပါ။ ရှုပ်နေမယ်ဆိုရင် အောက်မှာတဆင့်စီ ပြောပြပေးထားပါတယ်။

1. `.*`: `xhello123`
2. `.*h`: `xhello123` -> `xhello12`3 -> `xhello1`23 -> `xhello`123 -> `xhell`o123 -> `xhel`lo123 -> `xhe`llo123 -> `xh`ello123
3. `.*hello`: `xhello` 

ဒီ ဥပမာကို [DataCamp](https://www.datacamp.com/) ရဲ့ [Regular Expression in Python](https://campus.datacamp.com/courses/regular-expressions-in-python) ကနေယူထားတာပါ။ တကယ်လို့ Python နဲ့ data engineering ကို beginner အနေနဲ့ စလေ့လာမယ် ဆိုရင် [DataCamp](https://www.datacamp.com/) ကို recommend လုပ်ပါတယ်။ 
