+++
title = "Cursor Pagination"
date = '2025-01-10T22:30:00+07:00'
draft = false
lang = "my"
slug = "cursor-pagination"
tags = ["postgres", "mongodb", "sql", "nosql"]
+++

Mobile နဲ့ web app ရေးတဲ့သူတွေ၊ CURD API ရေးဖူးတဲ့သူတွေနဲ့ third-party API သုံးပြီးရေးဖူးတဲ့ programmer တွေဆိုရင် pagination ကို ရင်းရင်းနီးနီးသိကြမှာပါ။ ကိုယ်က API ရေးတဲ့သူဆိုရင် URL မှာ `limit` နဲ့ `skip` လိုမျိုး query parameter တွေထည့်ပေးတာမျိုး၊ validation အနေနဲ့ဆိုရင် `limit` မှာ အများဆုံး ဘယ်လောက်ပဲပေးထားတာမျိုး စသည်ဖြင့်ပေါ့။ Mobile developer တွေဆိုရင်လည်း [infinite scroll](https://en.wiktionary.org/wiki/infinite_scroll) လိုမျိုးတွေမှာ user က scroll လုပ်တာဘယ်နေရာကိုရောက်နေရင် နောက်တ page အတွက် API ကိုကြိုခေါ်ရမယ် ဘာညာပေါ့။ ဒါတွေက အားလုံးသိပြီးသားဖြစ်လောက်တဲ့အတွက် လိုရင်းကိုပဲသွားလိုက်ပါမယ်။

## Offset Pagination
Cursor pagination ဆိုပြီး ဘာလို့ `skip` နဲ့ `limit` သုံးတဲ့ pagination အကြောင်းအရင်ပြောရလဲဆိုရင် အသုံးအများဆုံး pagination ဖြစ်နေလို့ပါ။ သူအလုပ်လုပ်တဲ့ပုံစံက ရှင်းပါတယ်။ Query parameter အနေနဲ့ပြောရမယ်ဆို `?limit=20&skip=40` လို့ပေးလိုက်ရင် sort လုပ်ထားပြီးတဲ့အထဲကနေ ပထမ record ၄၀ ကို ဖယ်ပြီး record ၂၀ ပြန်ပေးလိုက်ရုံပါပဲ။

SQL အနေနဲ့ရေးရင်လည်း လွယ်လွယ်ကူကူ ရေးလို့ရပါတယ်။
```sql
SELECT *
FROM records
OFFSET 20
LIMIT 20;
```

MongoDB မှာဆိုရင်လည်း အဲ့လိုပါပဲ။
```javascript
db.records.find({}, { limit: 20, skip: 20 })
```

### Drawbacks of Offset Pagination
Offset pagination မှာအဓိက အားနည်းချက် နှစ်ခုရှိပါတယ်။

#### 1. Performance Issue on Large Dataset
ပထမတခုက performance issue ပါ။ Dataset ကအရမ်းကြီးလာပြီဆိုရင် `OFFSET` က ပြဿနာပေးလာပါတယ်။ လိုချင်တဲ့ data ကိုမထုတ်ပေးခင် table တခုလုံးကို scan လုပ်ဖို့လိုတဲ့အတွက်ပါ။ Query က complex ဖြစ်နေရင် ပိုတောင်ဆိုးပါသေးတယ်။ Database တခုထဲသုံးထားတာမျိုးဆို အဲ့လို query မျိုးကြောင့် system တခုလုံးရဲ့ performance ကိုပါ ထိခိုက်စေပါတယ်။ Microservices မှာဆိုရင်တော့ service တခုလုံးပေါ့။

#### 2. Inconsistent Data When Insert/Delete
ဒုတိယတခုက consistent ဖြစ်တဲ့ data ကိုမရတာပါ။ ဒီတခုက အပြောင်းအလဲသိပ်မများတဲ့ system မျိုးမှာဆိုရင် သိပ်မသိသာပါဘူး။ Social media app မျိုး၊ B2C နဲ့ C2C marketplace app မျိုးမှာဆိုရင် consistent ဖြစ်တဲ့ data ကို မပေးနိုင်တာက တကယ့်ပြဿနာပါ။

ဥပမာအနေနဲ့ပြောရရင် database table ထဲမှာ ID `1` ကနေ `100` အထိ record ၁၀၀ ရှိပြီး၊ descending order နဲ့ data ပြန်ပေးမယ်ဆိုကြပါစို့။ ပထမဆုံးအကြိမ်မှာ `OFFSET 0 LIMIT 20` နဲ့ data တောင်းလိုက်ရင် ID `100` ကနေ `81` အထိ record ၂၀ ရလာပါမယ်။ ဒုတိယအကြိမ်မှာ `OFFSET 20 LIMIT 20` နဲ့ data ထပ်တောင်းမှာဖြစ်တဲ့အတွက် ID `80` ကနေ `61` အထိ record ၂၀ ထပ်ရလာပါမယ်။ ဒီအထိ အားလုံးအဆင်ပြေနေပါသေးတယ်။

အဲ့ဒီမှာ ဒုတိယအကြိမ်မတိုင်ခင် ID `101` နဲ့ data အသစ်ထပ်ဝင်လာရင် ပြဿနာစပါပြီ။ `OFFSET 20 LIMIT 20` ဖြစ်တဲ့အတွက် ID `101` ကနေ `82` အထိ record ၂၀ ကိုကျော်လိုက်ပြီး ID `81` ကနေ `62` အထိ record ၂၀ ကို ပြန်ပေးလိုက်မှာပါ။ Infinite scroll မှာဆိုရင် ID `81` က နှစ်ခုပြပြီး ထပ်သွားပါပြီ။ QA တယောက်ထဲက စစ်နေရတဲ့ မျိုးမှာဆိုရင် testing အဆင့်မှာ မတွေ့ပဲကျော်သွားတတ်ပါတယ်။

Record တခုကို ဖျက်လိုက်ရင်လည်း အဲ့လိုပါပဲ။ ဒုတိယအကြိမ်မတိုင်ခင် ID `100` ကိုဖျက်လိုက်တယ် ဆိုကြပါစို့။ `OFFSET 20 LIMIT 20` နဲ့ data ယူလိုက်ရင် ID `79` ကနေ `60` အထိပဲပေးပြီး ID `80` က မပါလာတော့ပဲ ကျော်သွားမှာပါ။ ဒါက hard delete ပဲဖြစ်ဖြစ် `deleted_at` column သုံးတဲ့ soft delete ပဲဖြစ်ဖြစ် ကြုံရမယ့်ပြဿနာပါ။

## Cursor Pagination
Cursor pagination ကတော့ offset လိုမျိုးမဟုတ်ပဲ [ULID](https://github.com/ulid/spec) ဒါမှမဟုတ် `created_at` လိုမျိုး timestamp တွေကိုသုံးပြီး paginate လုပ်တာမျိုးပါ။

ဥပမာ SQL မှာဆိုရင် ဒီလိုမျိုးရေးလို့ရပါတယ်။
```sql
SELECT *
FROM records
WHERE created_at < 1736533304
LIMIT 20;
```

MongoDB မှာဆိုရင်တော့ အခုလိုမျိုးပေါ့။
```javascript
db.records.find(
  { created_at: { $lt: 1736533304 } }
).limit(20);
```

### Pros of Cursor Pagination
အခုလိုမျိုး condition-based ပုံစံ query ရေးလို့ရတဲ့အတွက် [offset pagination](#offset-pagination) မှာဖြစ်ခဲ့တဲ့ ပြဿနာတွေကို ဖြေရှင်းနိုင်သွားပါတယ်။

#### 1. Efficient Queries and Index Utilization
`OFFSET` ကို မသုံးရတော့တဲ့အတွက် table/collection တခုလုံးကို scan လုပ်စရာမလိုတော့တဲ့အတွက် query performance ပိုကောင်းလာပါမယ်။

Condition-based ပုံစံနဲ့ skip လုပ်တဲ့အတွက် pagination ကိုပါတွက်ပြီး index တွေဆောက်လို့ရသွားပါတယ်။ SQL ပဲဖြစ်ဖြစ် NoSQL ပဲဖြစ်ဖြစ် `OFFSET` အတွက် index ဆောက်လို့မရပါဘူး။

#### 2. Consistent Data When Insert/Delete
Offset pagination နဲ့ပြောင်းပြန် insert/delete ကြောင့် data တွေပျောက်သွားတာ၊ ထပ်နေတာတွေမရှိတဲ့၊ consistent ဖြစ်တဲ့ data ကိုရနိုင်ပါတယ်။ Insert/delete က ဘယ်လောက်ပဲ frequency မြင့်နေသည်ဖြစ်စေ data loss မဖြစ်နိုင်တော့ပါဘူး။

နောက်တခေါက်ထပ်ပြောရရင် data အသစ်ဝင်တာ၊ ဖျက်တာ ခဏခဏမရှိတဲ့ system မျိုးမှာဆိုရင် ဒါကိုထည့်မစဉ်းစားလည်း ရနိုင်ပါသေးတယ်။

### Drawbacks of Cursor Pagination
Cursor pagination ကလည်း silver bullet မဟုတ်တဲ့အတွက် အားနည်းချက်တွေရှိပါတယ်။

#### 1. Complexity
Cursor pagination က [offset pagination](#offset-pagination) ထက်စာရင် implement လုပ်ရတာ ပိုရှုပ်ပါတယ်။ ထည့်တွက်ရတာတွေလည်း ပိုများပါတယ်။ ဥပမာ `AUTO_INCREMENT` ID ကို မသုံးချင်ဘူး၊ `created_at` လို column မျိုးကိုလည်း အကြောင်းအမျိုးမျိုးကြောင့် မသုံးချင်ဘူးဆိုရင် UUID v7 လိုမျိုးကိုသုံးရမယ်၊ client ဘက်က request လုပ်လိုက်တဲ့ cursor က invalid ဖြစ်ပြီး data ထွက်မလာခဲ့ရင် business logic ပေါ်မူတည်ပြီး ဘယ် data ကို ပြန်ပေးမလဲ ထပ်ရေးရမယ်စသည်ဖြင့်ပေါ့။

#### 2. Inflexibility
အစဉ်လိုက်ပဲသွားလို့ အဆင်ပြေပါမယ်။ Page 1 ကနေ page 5 ကိုသွားမယ်ဆိုတာမျိုး လုပ်လို့မရပါဘူး။ မရဘူးဆိုတာထက် ရအောင်လို့ ကြံဖန်လုပ်ရမယ့်သဘောပါ။ ရခဲ့ရင်တောင် efficient မဖြစ်ပါဘူး။ နောက်တခုက ပြောင်းပြန်ပြန်သွားရခက်ပါတယ်။ ဥပမာ page 5 ကိုရောက်နေရာကနေ အရင်လာခဲ့တဲ့ page 4 ကိုပြန်သွားချင်တယ်ဆိုရင် မရပါဘူး။ Local storage လိုမျိုးမှာ အရှေ့ကသွားခဲ့တဲ့ page တွေရဲ့ data ကို မှတ်ထားမှရပါမယ်။

တခုရှိတာက infinite scroll မှာဆိုရင်တော့ ဒီပြဿနာတွေက မရှိသလောက်ပါပဲ။

#### 3. Depends on Sort and Order By
ဒီတခုက [inflexibility](#2-inflexibility) နဲ့ သွားဆက်စပ်ပါတယ်။ ဥပမာ page 5 လောက်ရောက်နေတုန်းမှာ name column လိုမျိုးနဲ့ ပြောင်းပြီး sort လုပ်မယ်ဆိုတာမျိုး၊ ascending ကနေ descending order ပြောင်းမယ်ဆိုတာမျိုး လုပ်လို့မရပါဘူး။ ရအောင်လုပ်လည်း efficient မဖြစ်ပါဘူး။ ကပ်သီးကပ်သပ် ရအောင်လုပ်နိုင်တာတွေရှိလို့ ကြိုပြောထားရတာပါ။

#### 4. Misc
နောက်တခုက ဖြစ်နိုင်ချေနည်းပေမဲ့ ထည့်စဉ်းစားထားသင့်ပါတယ်။ Sort လုပ်ဖို့သုံးမယ့် column တွေက ထပ်လို့မရပါဘူး။ ID လိုမျိုးမှာတော့ကိစ္စမရှိပေမယ့် `created_at` column မှာဆိုရင် millisecond အထိထပ်သွားနိုင်ချေရှိနေပါတယ်။

## Conclusion
အနှစ်ချုပ်အနေနဲ့ပြောရရင် ဘာလာလာ ဒါပဲသုံးမယ်ဆိုတဲ့ silver bullet solution မျိုးမရှိပါဘူး။ System ရဲ့ သဘာဝပေါ်မူတည်ပြီး သင့်တော်တာကိုသုံးသွားရမှာပါ။

ဒီ API က mobile မှာပဲသုံးမှာမလို့ [cursor pagination](#cursor-pagination) ပဲသုံးလိုက်မယ်လုပ်ရင် web version ထုတ်ချင်တဲ့အချိန်ကျ တိုင်ပတ်ပါလိမ့်မယ်။ Insert/delete က သိပ်မရှိပါဘူးဆိုပြီး [offset pagination](#offset-pagination) သုံးထားရင်လည်း frequency များလာရင် ဒုက္ခများမှာပါ။

Consumer app လိုမျိုးမှာဆိုရင် force update လုပ်ဖို့က ထင်သလောက်မလွယ်ပါဘူး။ User base ကြီးရင်ကြီးသလောက် အများကြီးစဉ်းစားရပါတယ်။ ဒါမျိုးကြုံလာခဲ့ရင်လည်း API version ခွဲထုတ်ပေးတာမျိုးနဲ့ ဖြေရှင်းလို့ရနိုင်ပါလိမ့်မယ်။
