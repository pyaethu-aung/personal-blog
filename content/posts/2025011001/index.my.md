+++
title = "Cursor Pagination"
date = '2025-01-10T22:30:00+07:00'
draft = true
lang = "my"
slug = "cursor-pagination"
tags = ["postgres", "mongodb", "sql", "nosql"]
+++

Mobile နဲ့ web app ရေးတဲ့သူတွေ၊ CURD API ရေးဖူးတဲ့သူတွေနဲ့ third-party API သုံးပြီးရေးဖူးတဲ့ programmer တွေဆိုရင် pagination ကို ရင်းရင်းနီးနီးသိကြမှာပါ။ ကိုယ်က API ရေးတဲ့သူဆိုရင် URL မှာ `limit` နဲ့ `skip` လိုမျိုး query parameter တွေထည့်ပေးတာမျိုး၊ validation အနေနဲ့ဆိုရင် `limit` မှာ အများဆုံး ဘယ်လောက်ပဲပေးထားတာမျိုးတွေ စသဖြင့်ပေါ့။ Mobile programmer တွေဆိုရင်လည်း infinite scroll လိုမျိုးတွေမှာ user က scroll လုပ်တာဘယ်နေရာကိုရောက်နေရင် နောက်တ page အတွက် API ကိုကြိုခေါ်ရမယ် ဘာညာပေါ့။ ဒါတွေက အားလုံးသိပြီးသားဖြစ်လောက်တဲ့အတွက် လိုရင်းကိုပဲသွားလိုက်ပါမယ်။
