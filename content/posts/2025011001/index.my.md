+++
title = "Cursor Pagination"
date = '2025-01-10T22:30:00+07:00'
draft = true
lang = "my"
slug = "cursor-pagination"
tags = ["postgres", "mongodb", "sql", "nosql"]
+++

Mobile နဲ့ web app ရေးတဲ့သူတွေ၊ CURD API ရေးဖူးတဲ့သူတွေနဲ့ third-party API သုံးပြီးရေးဖူးတဲ့ programmer တွေဆိုရင် pagination ကို ရင်းရင်းနီးနီးသိကြမှာပါ။ ကိုယ်က API ရေးတဲ့သူဆိုရင် URL မှာ `limit` နဲ့ `skip` လိုမျိုး query parameter တွေထည့်ပေးတာမျိုး၊ validation အနေနဲ့ဆိုရင် `limit` မှာ အများဆုံး ဘယ်လောက်ပဲပေးထားတာမျိုးတွေ စသဖြင့်ပေါ့။ Mobile programmer တွေဆိုရင်လည်း infinite scroll လိုမျိုးတွေမှာ user က scroll လုပ်တာဘယ်နေရာကိုရောက်နေရင် နောက်တ page အတွက် API ကိုကြိုခေါ်ရမယ် ဘာညာပေါ့။ ဒါတွေက အားလုံးသိပြီးသားဖြစ်လောက်တဲ့အတွက် လိုရင်းကိုပဲသွားလိုက်ပါမယ်။

## Offset Pagination
Cursor pagination ဆိုပြီး ဘာလို့ `skip` နဲ့ `limit` သုံးတဲ့ pagination အကြောင်းအရင်ပြောရလဲဆိုရင် အသုံးအများဆုံး pagination ဖြစ်နေလို့ပါ။ သူအလုပ်လုပ်တဲ့ပုံစံက ရှင်းပါတယ်။ Query parameter အနေနဲ့ပြောရမယ်ဆို `?limit=20&skip=40` လို့ပေးလိုက်ရင် sort လုပ်ထားပြီးတဲ့အထဲကနေ ပထမ record ၄၀ ကို ဖယ်ပြီး record ၂၀ ပြန်ပေးလိုက်ရုံပါပဲ။

SQL အနေနဲ့ရေးရင်လည်း လွယ်လွယ်ကူကူ ရေးလို့ရပါတယ်။
```sql
SELECT *
FROM records
OFFSET 20
LIMIT 20
```
MongoDB မှာဆိုရင်လည်း အဲလိုပါပဲ။
```javascript
db.records.find({}, { limit: 20, skip: 20 })
```

### Drawbacks of Offset Pagination
Offset pagination မှာအဓိက အားနည်းချက် နှစ်ခုရှိပါတယ်။

#### Performance Issue on Large Dataset
ပထမတခုက performance issue ပါ။ Dataset ကအရမ်းကြီးလာပြီဆိုရင် `OFFSET` က ပြဿနာပေးလာပါတယ်။ လိုချင်တဲ့ data ကိုမထုတ်ပေးခင် table တခုလုံးကို scan လုပ်ဖို့လိုတဲ့အတွက်ပါ။ Query က complex ဖြစ်ရင် ပိုတောင်ဆိုးပါသေးတယ်။ Database တခုထဲသုံးထားတာမျိုးဆို အဲလို query မျိုးကြောင့် system တခုလုံးရဲ့ performance ကိုပါ ထိခိုက်စေပါတယ်။ Microservices မှာဆိုရင်တော့ service တခုလုံးပေါ့။

#### Inconsistent Data When Insert/Remove
ဒုတိယတခုက consistent ဖြစ်တဲ့ data ကိုမရတာပါ။ ဒီတခုက အပြောင်းအလဲသိပ်မများတဲ့ system မျိုးမှာဆိုရင် သိပ်မသိသာပါဘူး။ Social media app မျိုး၊ B2C နဲ့ C2C marketplace app မျိုးမှာဆိုရင် consistent ဖြစ်တဲ့ data ကို မပေးနိုင်တာက တကယ့်ပြဿနာပါ။

ဥပမာအနေနဲ့ပြောရရင် database table ထဲမှာ ID `1` ကနေ `100` အထိ record ၁၀၀ ရှိပြီး၊ descending order နဲ့ data ပြန်ပေးမယ်ဆိုကြပါစို့။ ပထမဆုံးအကြိမ်မှာ `OFFSET 0 LIMIT 20` နဲ့ data တောင်းလိုက်ရင် ID `100` ကနေ `81` အထိ record ၂၀ ရလာပါမယ်။ ဒုတိယအကြိမ်မှာ `OFFSET 20 LIMIT 20` နဲ့ data ထပ်တောင်းမှာဖြစ်တဲ့အတွက် ID `61` ကနေ `80` အထိ record ၂၀ ထပ်ရလာပါမယ်။ ဒီအထိ အားလုံးအဆင်ပြေနေပါသေးတယ်။

အဲဒီမှာ ဒုတိယအကြိမ်မတိုင်ခင် ID `101` နဲ့ data အသစ်ထပ်ဝင်လာရင်  ပြဿနာစပါပြီ။ `OFFSET 20 LIMIT 20` ဖြစ်တဲ့အတွက် ID `101` ကနေ `82` အထိ record ၂၀ ကိုကျော်လိုက်ပြီး ID `81` ကနေ `62` အထိ record ၂၀ ကို ပြန်ပေးလိုက်မှာပါ။ Infinite scroll မှာဆိုရင် ID `81` က နှစ်ခုပြပြီး ထပ်သွားပါပြီ။
