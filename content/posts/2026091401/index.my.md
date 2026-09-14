+++
title = "ဘာလို့ Parquet File က Space ပိုနည်း Query ပိုမြန်ရတာလဲ"
date = '2026-09-14T23:31:00+07:00'
draft = false
slug = "parquet-file-format"
tags = ["parquet", "data-engineering", "big-data", "database"]
+++

[Read this article in English]({{< relref path="2026091401/index.en.md" lang="en" >}})

## Introduction

Data အပိုင်းမှာ အလုပ်လုပ်နေသူတိုင်း Parquet file တွေက CSV ထက် file အရွယ်အစား အများကြီးသေးတယ်၊ Query performance လည်း ပိုမြန်တယ်ဆိုတာ သိနေကြမှာပါ။ ကျွန်တော်တို့က Parquet က ပိုကောင်းတယ်ဆိုတာ သိကြပေမဲ့ **ဘာလို့ပိုကောင်းတာလဲ** ဆိုတာကိုတော့ သိတဲ့သူနည်းပါလိမ့်မယ်။

ဒီပိတ်ရက်မှာ Michael Berk ရဲ့ [Demystifying the Parquet File Format](https://towardsdatascience.com/demystifying-the-parquet-file-format-13adb0206705/) ဆိုတဲ့ article ဖတ်မိလိုက်တာမှာ S3 ပေါ်က 1 TB data နဲ့ စမ်းပြထားတာ တွေ့မိလိုက်ပါတယ်။ Parquet ဟာ CSV ထက် **Storage 87% သက်သာသွားတယ်** Query Speed **34 ဆ ပိုမြန်သွားတယ်** ဆိုတာ တော်တော်ကိုသိသာတဲ့ ရလဒ်ပါ။

ဒီ Post မှာတော့ Parquet file ရဲ့ architecture layout၊ storage သက်သာရတဲ့ အကြောင်းအရင်း၊ query ပိုမြန်ရတဲ့ အကြောင်းအရင်းနဲ့ Parquet ကို မသုံးသင့်တဲ့ အခြေအနေတွေကို မျှဝေပေးသွားပါမယ်။

## What Problem Does Parquet Solve?

Table တစ်ခုမှာ row နဲ့ column ဆိုပြီး dimension နှစ်ခု ရှိပါတယ်။ Disk ပေါ်က file ကတော့ dimension တခုပဲရှိပြီး sequence of bytes တွေပါ။ ဒါကြောင့် file format တခုကို storage တခုပေါ်မှာသိမ်းတိုင်း အဓိကဖြေရှင်းရမှာက **Cell တွေကို disk ပေါ်မှာ ဘယ်လို order နဲ့ သိမ်းရမလဲ** ဆိုတာပါ။

Dashboards၊ reports နဲ့ data science လို analytics workload (OLAP) တွေမှာ အဓိကစဉ်းစားရတဲ့ အချက် နှစ်ချက်ရှိပါတယ်။

- **Read speed**: လိုအပ်တဲ့ data ကို ဘယ်လောက်မြန်မြန်ရှာပြီး decode လုပ်နိုင်သလဲ။
- **Size on disk**: Disk ပေါ်မှာ ဘယ်လောက်နေရာယူထားသလဲ။

Write speed လည်း အရေးကြီးပေမဲ့ analytics data တွေက များသောအားဖြင့် write တကြိမ်ပဲ လုပ်ပြီး read ကို အကြိမ်ကြိမ် လုပ်ကြတာပါ။

## Row-Based vs Column-Based vs Hybrid

ဥပမာအနေနဲ့ ဒီ `orders` table sample ကို ကြည့်ပါ။

| id       | country | status  | amount |
|----------|---------|---------|--------|
| 1        | MM      | PAID    | 25.00  |
| 2        | MM      | PAID    | 12.50  |
| 3        | TH      | PENDING | 40.00  |
| 4        | MM      | PAID    | 8.00   |

ဒီ table data ကို file တခုအနေနဲ့သိမ်းဖို့ နည်းလမ်း သုံးခုရှိပါတယ်။

### 1. Row-based (CSV)

Row တခုပြီးတခု အစဉ်လိုက်သိမ်းတာပါ။

```text
1,MM,PAID,25.00 | 2,MM,PAID,12.50 | 3,TH,PENDING,40.00 | 4,MM,PAID,8.00
```

### 2. Column-based

Column တခုပြီးတခု အစဉ်လိုက်သိမ်းတာပါ။

```text
1,2,3,4 | MM,MM,TH,MM | PAID,PAID,PENDING,PAID | 25.00,12.50,40.00,8.00
```

### 3. Hybrid (Parquet)

ပထမဆုံးအနေနဲ့ row တွေကို **row groups** ဆိုပြီး အုပ်စုဖွဲ့လိုက်ပါတယ်။ ပြီးမှ row group တခုစီမှာ data တွေကို column အလိုက်သိမ်းတာပါ။ Row group size ကို `2` လို့ သတ်မှတ်ထားရင် ဒီလိုမြင်ရပါမယ်။

```text
┌──────────────── Row Group 0 (rows 1–2) ────────────────┐
│ 1,2 │ MM,MM │ PAID,PAID │ 25.00,12.50                  │
└────────────────────────────────────────────────────────┘
┌──────────────── Row Group 1 (rows 3–4) ────────────────┐
│ 3,4 │ TH,MM │ PENDING,PAID │ 40.00,8.00                │
└────────────────────────────────────────────────────────┘
```

Parquet ကို များသောအားဖြင့် **columnar** format လို့ ခေါ်ကြပေမဲ့ **column chunk** တွေအနေနဲ့ သိမ်းထားတာကြောင့် "hybrid" လို့ ခေါ်တာက ပိုမှန်မှာပါ။

ဒီလိုသိမ်းတာက ဘာလို့အရေးကြီးတာလဲဆိုရင် analytics query တွေမှာက အဓိကနှစ်ပိုင်းရှိပါတယ်။

- **Projection**: လိုချင်တဲ့ column တွေကို ရွေးထုတ်တာပါ (SQL ရဲ့ `SELECT` အပိုင်း)။
- **Predicate**: လိုချင်တဲ့ row တွေကို စစ်ထုတ်တာပါ (SQL ရဲ့ `WHERE` အပိုင်း)။

Column layout က လိုအပ်တဲ့ column တွေကိုပဲ သီးသန့်ရွေးယူနိုင်တာကြောင့် projection အတွက် အရမ်းကောင်းပါတယ်။ Row group တွေကတော့ predicate filters တွေပေါ် မူတည်ပြီး မလိုတဲ့ row group တခုလုံးကို skip လုပ်လို့ရပါတယ်။ Hybrid layout က ဒီနှစ်ချက်လုံးကို support ပေးထားတဲ့အတွက် analytics workloads တွေအတွက် အရမ်းအဆင်ပြေစေပါတယ်။

## What Is Inside a Parquet File?

Parquet dataset တခုရဲ့ ဖွဲ့စည်းပုံကို အဆင့်လိုက်ကြည့်ရင် ဒီလိုရှိမှာပါ။

```text
Root directory
└── .parquet files
    └── Row groups
        └── Column chunks
            └── Data pages
```

Single `.parquet` file တစ်ခုရဲ့ architecture ကတော့ ဒီလိုပါ။

```text
┌──────────────────────────────────────────────┐
│ "PAR1"  (4-byte magic number)                │
├──────────────────────────────────────────────┤
│ Row Group 0                                  │
│   ├── Column Chunk: id                       │
│   │     ├── Dictionary Page (optional)       │
│   │     ├── Data Page 0                      │
│   │     ├── Data Page 1                      │
│   │     └── ... (one or more pages)          │
│   ├── Column Chunk: country                  │
│   ├── Column Chunk: status                   │
│   └── Column Chunk: amount                   │
├──────────────────────────────────────────────┤
│ Row Group 1 ... Row Group N                  │
├──────────────────────────────────────────────┤
│ Footer (File Metadata)                       │
│   ├── schema                                 │
│   ├── where each column chunk starts         │
│   └── statistics (min, max, null count)      │
├──────────────────────────────────────────────┤
│ Footer length (4 bytes)                      │
│ "PAR1"  (4-byte magic number)                │
└──────────────────────────────────────────────┘
```

- **Row group**: Table ရဲ့ horizontal slice ဖြစ်ပါတယ်။ (ဥပမာ row ပေါင်း ၁၀၀,၀၀၀)။
- **Column chunk**: Row group တခုမှာရှိတဲ့ column တခုရဲ့ တန်ဖိုးတွေပါ။
- **Data page**: အသေးဆုံး Unit ပါ။ Actual encoded value တွေကို သိမ်းထားတာပါ။ Column chunk တခုကို data page အများကြီး (Default 1 MB ခန့်စီ) ခွဲထားတာကြောင့် reader ဘက်က လိုအပ်သလို တ page စီ decompress သို့မဟုတ် skip လုပ်နိုင်ပါတယ်။
- **Footer**: File ရဲ့ metadata တွေပါ။ Data write ပြီးမှ offset နဲ့ statistic တွေကို တိတိကျကျသိနိုင်မှာဖြစ်တာကြောင့် footer ကို file ရဲ့အဆုံးမှာ ရေးထားတာပါ။

ပြန် read ရင်ကျတော့ ပြောင်းပြန်ပါ။ နောက်ဆုံး 8 bytes ကို အရင်ဖတ်၊ footer ကိုဖတ်ပြီးတာနဲ့ file တခုလုံးရဲ့ map ကို data တွေ လိုက်မဖတ်ခင်မှာပဲ ရသွားတာပါ။

## Why Parquet Takes Less Space

### 1. Binary types instead of text

CSV မှာ data အားလုံးကို text အနေနဲ့သိမ်းပါတယ်။ `1234567890` ဆိုတဲ့ ကိန်းဂဏန်းက text အနေနဲ့ဆို 10 bytes ယူပါတယ်။ Parquet မှာတော့ `INT32` အနေနဲ့ သိမ်းတာကြောင့် 4 bytes ပဲ ယူပါတယ်။ `2026-09-14T22:00:00Z` လို timestamp က text နဲ့ဆို 20 bytes ယူပေမဲ့ `INT64` နဲ့ဆို 8 bytes ပဲ ယူပါတယ်။ ဘာမှ compress မလုပ်ရသေးခင်မှာတင် space ပိုသက်သာနေပါပြီ။

### 2. Run-length encoding (RLE)

ဥပမာ column တခုမှာ value ပေါင်း ၁၀,၀၀၀,၀၀၀ ပါပြီး အားလုံးက `0` တွေကြီးပဲ ဖြစ်နေတယ်ဆိုပါတော့။ `0` ပေါင်း ၁၀ သန်း မသိမ်းတော့ပဲ value နဲ့ frequency နံပါတ်နှစ်ခုပဲ သိမ်းလိုက်ပါတယ်။

```text
Raw:   0 0 0 0 0 0 0 ... (10,000,000 times)

RLE:   (value = 0, count = 10,000,000)
```

ကိန်းဂဏန်းပေါင်း ၉,၉၉၉,၉၉၈ ခု သိမ်းရမယ့် space သက်သာသွားတာပါ။ Real-world data တွေမှာ ဒီလိုထပ်နေတာမျိုးက သိပ်မရှိနိုင်ပေမယ့်  sorted `status` သို့မဟုတ် sorted `country` လို column တွေမှာ RLE က အတော်လေး ထိရောက်မှုရှိပါတယ်။

### 3. Dictionary encoding with bit-packing

နိုင်ငံနာမည်တွေလို ထပ်ခါထပ်ခါ ပါနေတတ်တဲ့ long string တွေကို သိမ်းရတာမျိုးမှာလည်း optimize လုပ်ထားပါတယ်။။ **The Democratic Republic of Congo** (32 characters) လို string မျိုးကို string အတိုင်း အကြိမ်ကြိမ်သိမ်းရင် space အများကြီးကုန်မှာပါ။

Dictionary encoding က တူညီတဲ့ string value တွေကို တကြိမ်ပဲသိမ်းပြီး value နေရာတွေမှာ integer တွေနဲ့ အစားထိုးလိုက်ပါတယ်။

```text
Raw values:   MM  MM  TH  MM  MM  SG  TH  MM

Dictionary:   0 → MM
              1 → TH
              2 → SG

Indexes:      0   0   1   0   0   2   1   0
```

အဲဒီ integer တွေကိုမှ **bit-packing** ထပ်လုပ်ပါတယ်။ နိုင်ငံသုံးခုပဲရှိတာကြောင့် index တစ်ခုစီအတွက် 4-byte integer မလိုဘဲ 2 bits ပဲ သုံးပါတယ်။

Row ၁၀ သန်းမှာ distinct country 200 ပါဝင်ပြီး တစ်ခုကို average 10 bytes ရှိတယ်လို့ထားပြီး တွက်ကြည့်နိုင်ပါတယ်။

```text
Plain strings:   10,000,000 × ~10 bytes  ≈ 100 MB
Dictionary:      200 × ~10 bytes         ≈   2 KB
Indexes:         10,000,000 × 8 bits     ≈  10 MB   (200 values fit in 8 bits)
```

Compression မလုပ်ရသေးခင်မှာတင် ၁၀ ဆမက သက်သာသွားတာကို တွေ့ရမှာပါ။

### 4. Delta encoding

ID နဲ့ timestamp လိုမျိုး incremental data type တွေမှာ value အပြည့်အစုံသိမ်းနေမယ့်အစား အရှေ့ or အနောက်က value နဲ့ ခြားနားချက် (Delta) ကို သိမ်းတာက space ပိုသက်သာပါတယ်။

```text
Timestamps:  1757858400  1757858401  1757858403  1757858404

Delta:       1757858400  +1  +2  +1
```

ခြားနားချက်လေးတွေက သေးငယ်တာကြောင့် bit-packing လုပ်ရတာ လွယ်ကူသွားပါတယ်။

### 5. Compression on top of encoding

Encoding လုပ်ပြီးတဲ့နောက်မှာ page တစ်ခုစီကို **Snappy**၊ **GZIP**၊ **ZSTD** စတဲ့ codec တွေနဲ့ compression ထပ်လုပ်ပါတယ်။

```text
Column values
     │
     ▼
Encoding      (RLE, dictionary, bit-packing, delta)
     │
     ▼
Compression   (Snappy, GZIP, ZSTD)
     │
     ▼
Bytes on disk
```

Compression ဆိုတာမျိုးက data အမျိုးအစား တူညီတဲ့အခါ ပိုပြီးထိထိရောက်ရောက် compress လုပ်နိုင်ပါတယ်။ Data type တူတဲ့ column တခုတည်းကို compress လုပ်ရတာကြောင့် သက်သာတာပါ။ CSV ကို GZIP အနေနဲ့ compress လုပ်ရင်တော့ number, string နဲ့ commas တွေ ရောထွေးနေတာကြောင့် ထိရောက်မှုကတော့ ပိုနည်းပါလိမ့်မယ်။

### 6. Nulls cost almost nothing

Parquet က null cell တွေကို value အနေနဲ့ သီးသန့်မသိမ်းပါဘူး။ Data ရှိမရှိပြောနိုင်တဲ့ **definition levels** နံပါတ်အသေးတွေကိုပဲ RLE-encoded လုပ်ပြီး သိမ်းပါတယ်။ Repetition level ပါ တွဲသုံးပြီး nested data (list or struct) တွေကို ပါ column အလိုက် သိမ်းနိုင်အောင် လုပ်ပေးထားတာပါ။

## Why Parquet Queries Are Fast

Column ၄၀ ပါတဲ့ `orders` table ကို ဒီ SQL query နဲ့ ဥပမာ ပေးပါမယ်။

```sql
SELECT country, SUM(amount)
FROM orders
WHERE order_date = '2026-09-01'
GROUP BY country;
```

### 1. Projection: read only the columns you need

Query က column အခု ၄၀ ထဲက သုံးခုပဲလိုတာဆိုတော့ query engine က column chunk သုံးခုကိုပဲ ဖတ်ပါတယ်။ ကျန်တဲ့ column ၃၇ ခုအတွက် disk or S3 ကနေ file တခုလုံး download ဆွဲစရာ မလိုတော့ပါဘူး။

```text
              id   country   status   amount   order_date   ...35 more
Row Group 0    ✗         ✓        ✗        ✓            ✓            ✗
Row Group 1    ✗         ✓        ✗        ✓            ✓            ✗
Row Group 2    ✗         ✓        ✗        ✓            ✓            ✗
```

CSV ဘက်မှာကျ column တခုတည်း လိုချင်ရင်တောင် Row တိုင်း ရဲ့ byte မှန်သမျှကို read/parse လုပ်ရပါတယ်။

### 2. Predicate pushdown: skip row groups with min/max statistics

Footer ထဲမှာ row group တခုစီမှာရှိနေတဲ့ column တိုင်း ရဲ့ **min/max** statistics တွေကို သိမ်းထားပါတယ်။ Engine က data တွေကို အရင်မဖတ်ခင် `WHERE` clause နဲ့ အဲဒီ stats တွေကို အရင် query လုပ်ကြည့်ပါတယ်။

```text
WHERE order_date = '2026-09-01'

Row Group 0   min=2026-08-01  max=2026-08-15   → skip
Row Group 1   min=2026-08-16  max=2026-08-31   → skip
Row Group 2   min=2026-09-01  max=2026-09-14   → READ
Row Group 3   min=2026-09-15  max=2026-09-30   → skip
```

Row group လေးခုထဲက သုံးခုကို လုံးဝ parse လုပ်စရာမလိုပဲ skip သွားတာကို တွေ့ရမှာပါ။

သတိထားရမှာက data တွေကို filter လုပ်မည့် column အလိုက် **sorted** ဒါမှမဟုတ် **clustered** လုပ်ထားမှ ဒီ feature က အလုပ်လုပ်မှာပါ။ Random ဖြစ်နေတယ်ဆိုရင် row group တိုင်းမှာ min/max တွေက ပြန့်နေပြီး skip လုပ်နိုင်မှာ မဟုတ်ပါဘူး။

ဒီနည်းလမ်းကို file level မှာလည်း သုံးပါတယ်။ Directory တစ်ခုထဲမှာ `.parquet` file အများကြီးရှိနေရင် file metadata ကို ကြည့်ပြီး file အလိုက်ပါ skip လုပ်သွားနိုင်ပါတယ်။

### 3. Reading the footer first

Metadata အားလုံးက footer ထဲမှာဆိုတော့ query engine က ဘယ် files၊ row group၊ column တွေကို parse လုပ်ရမလဲဆိုတာ ကြိုစီစဉ်လို့ရသွားပါတယ်။ S3 ပေါ်ကနေဆိုရင် file တခုလုံး download ဆွဲစရာမလိုဘဲ လိုအပ်တဲ့ byte ranges နည်းနည်းကိုပဲ fetch လုပ်လို့ရသွားပါတယ်။

```text
1. Read last 8 bytes         → footer length
2. Read footer               → schema, offsets, min/max
3. Decide what to skip       → list of byte ranges
4. Fetch only those ranges   → decode → return result
```

### 4. Lazy evaluation in Spark

Spark လို engine တွေမှာ operation တွေက lazy evaluation နဲ့ပါ။ Result ကို အမှန်တကယ်လိုအပ်လာမှပဲ run တာကြောင့် query တခုလုံးကို ကြိုကြည့်၊ projection နဲ့ predicate တွေကို Parquet reader ဆီ push down လုပ်ပေးနိုင်သွားပါတယ်။ Data တွေကို memory ပေါ်တင်ပြီးမှ filter လုပ်တာ မဟုတ်ဘဲ read လုပ်နေတုန်းမှာပဲ filter လုပ်သွားတာပါ။

### 5. Parallel and vectorized reads

Row group တွေက independent ဖြစ်နေတဲ့အတွက် thread အများကြီးကနေ ဒါမှမဟုတ် machine အများကြီးကနေ တပြိုင်နက်တည်း ဖတ်လို့ရပါတယ်။။ Column chunk တွေကလည်း data type တူတဲ့ packed array တွေဖြစ်တဲ့အတွက် Spark, DuckDB, Arrow စတဲ့ engine တွေကပါ batch အလိုက် process လုပ်နိုင်သွားတဲ့အတွက် CPU အတွက်ဆိုရင် row အလိုက် process လုပ်ရတာထက် အများကြီး ပိုမြန်သွားပါတယ်။

## Trying It Yourself with DuckDB

[DuckDB](https://duckdb.org/) သုံးပြီးလည်း CSV ကနေ Parquet ကို အလွယ်တကူ စမ်းပြောင်းကြည့်လို့ရပါတယ်။

```sql
COPY (SELECT * FROM 'orders.csv' ORDER BY order_date)
TO 'orders.parquet' (FORMAT parquet, COMPRESSION zstd);
```

`ORDER BY` ထည့်ထားတာကို သတိထားကြည့်ပါ။ Filter လုပ်မယ့် column ကို sort လုပ်ထားခြင်းအားဖြင့် min/max skipping နဲ့ RLE တို့ကို ပိုပြီး ထိထိရောက်ရောက်သုံးနိုင်မှာပါ။

Parquet file metadata ကို ဒီ query နဲ့ စစ်ကြည့်ပါ။

```sql
-- Row groups, encodings, compression, and min/max statistics
SELECT row_group_id, path_in_schema, encodings, compression,
       stats_min, stats_max, total_compressed_size
FROM parquet_metadata('orders.parquet');

-- Schema stored in the footer
SELECT * FROM parquet_schema('orders.parquet');
```

`orders.csv` နှင့် `orders.parquet` file နှစ်ခုရဲ့ file size ကွာခြားချက်ကို ယှဉ်ကြည့်ပါ။

## Quick Comparison with CSV

| | CSV | Parquet |
|---|---|---|
| Layout | Row-based | Hybrid (row groups of columns) |
| Data types | Everything is text | Binary types, schema in the file |
| Size on disk | Large | Small (encoding + compression) |
| Read only some columns | No, must parse every row | Yes |
| Skip rows with filters | No | Yes, using min/max statistics |
| Human-readable | Yes | No, you need a tool |
| Append / edit rows | Easy | Rewrite the file |

## When Parquet Is Not a Good Fit

Parquet က analytics အတွက်ရည်ရွယ်ပြီး design လုပ်ထားတဲ့အတွက် trade-off တွေလည်း ရှိပါတယ်။

- **Files are immutable**: Single row တခုတည်းကိုကွက်ပြီး update သွားလုပ်လို့ မရပါဘူး။ Row update လုပ်ချင်ရင် file တခုလုံးကို ပြန်ရေးရမှာပါ။
- **Not for point lookups or OLTP**: Select row by ID ဆိုတာမျိုးနဲ့ မသင့်တော်ပါဘူး။
- **Small Files Problem**: File သေးသေးလေးတွေ အများကြီးဖြစ်နေရင် နှေးပါတယ်။ Parquet writer တွေက file အများကြီး ထုတ်ပေးတတ်တဲ့အတွက် dataset သေးသေးလေးတွေဆိုရင် မရေးခင် repartition အရင်လုပ်သင့်ပါတယ်။
- **Writing speed နှေးပါတယ်**: Memory ထဲမှာ row group အလိုက် buffer လုပ်ရတာ၊ dictionary ဆောက်ရတာနဲ့ compress လုပ်ရတာတွေကြောင့်ပါ။
- **Human-readable မဖြစ်ပါဘူး**: Terminal မှာ `cat` command နဲ့ အမြန်ကြည့်ချင်တာမျိုး မရပါဘူး။
- **`SELECT *` benefits less.**: column အကုန်လုံးဖတ်တဲ့ query မျိုးဆိုရင် projection ရဲ့ advantages တွေ ရမှာမဟုတ်ပါဘူး။

## What About Updates and Transactions?

Parquet file တွေရဲ့ immutability ပြဿနာကို ဖြေရှင်းဖို့ [Delta Lake](https://delta.io/) လို storage framework တွေကိုလည်း သုံးကြပါတယ်။ Delta Lake က Parquet file တွေပေါ်မှာ transaction log တခု ထပ်အုပ်ပြီး **ACID transaction**၊ update နဲ့ delete transaction တွေကို support လုပ်ပေးသွားတာပါ။။ [Apache Iceberg](https://iceberg.apache.org/) ကိုလည်း အဲ့အတွက်သုံးကြလေ့ရှိပါတယ်။

## Practical Tips

- Filter အများဆုံး လုပ်မယ့် column တွေကို **sort** လုပ်ထားပါ။
- သင့်တော်တဲ့ **row group size** ကို ရွေးပါ။
- Storage/Network ကုန်ကျစရိတ် ပိုသက်သာချင်ရင် **ZSTD** ကို သုံးပါ (Snappy က ပိုမြန်ပေမယ့် file size ပိုကြီးပါတယ်။)။
- Large dataset တွေကို Date လို data type မျိုးနဲ့ **partition** ပိုင်းထားပါ။
- ဖိုင်သေးသေးလေးတွေ အများကြီးဖြစ်မနေအောင် ပုံမှန် **compaction** လုပ်ပေးပါ။
- ဖြစ်နိုင်ရင် proper data type တွေ (ဥပမာ date အတွက် ကို String အစား `DATE`/`TIMESTAMP`) သုံးပါ။

## Conclusion

Parquet က storage သက်သက်သာသာနဲ့ query time မြန်မြန်ရဖို့ **chunks of columns** နဲ့ **metadata** တွေကို ထိထိရောက်ရောက်ပေါင်းစပ်ထားတာပါ။။

Data ပမာဏ များပြီး column အနည်းကိုပဲ လုပ်တာများတဲ့ analytics workload တွေမှာ Parquet က အကောင်းဆုံး default choice တခုပါ။ ဒီ article မှာ ကိုးကားထားတဲ့ Michael Berk ရဲ့ [မူရင်းဆောင်းပါး](https://towardsdatascience.com/demystifying-the-parquet-file-format-13adb0206705/) ကိုလည်း ဖတ်ကြည့်ဖို့ တိုက်တွန်းပါတယ်။