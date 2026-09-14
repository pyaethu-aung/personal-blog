+++
title = "Why Parquet Files Take Less Space and Query Faster"
date = '2026-09-14T23:31:00+07:00'
draft = false
slug = "parquet-file-format"
tags = ["parquet", "data-engineering", "big-data", "database"]
+++

[မြန်မာဘာသာဖြင့် ဖတ်ရှုရန်]({{< relref path="2026091401/index.my.md" lang="my" >}})

## Introduction

If you have worked with data, you have probably heard that Parquet files are much smaller than CSV and much faster to query. Many of us know that Parquet is **better**, but not really **why**.

Recently, I read [Demystifying the Parquet File Format](https://towardsdatascience.com/demystifying-the-parquet-file-format-13adb0206705/) by Michael Berk. The article cites a comparison on 1 TB of data stored in S3 where Parquet takes **87% less space** and queries **34x faster** than CSV. That is a huge difference for just changing a file format.

In this post, I will share how a Parquet file is laid out, why it takes less space, why queries on it are fast, and when Parquet is not the right choice.

## What Problem Does Parquet Solve?

A table has two dimensions: rows and columns. A file on disk is just a one-dimensional sequence of bytes. So every file format has to decide one thing: **in what order do we write the cells?**

For analytics workloads (OLAP), like dashboards, reports, and data science, analytics workloads focus on two things::

- **Read speed**: how fast we can find and decode the data we need.
- **Size on disk**: how many bytes the file takes.

Write speed matters too, but analytics data is usually written once and read many times.

## Row-Based vs Column-Based vs Hybrid

Let's use this small `orders` table:

| id       | country | status  | amount |
|----------|---------|---------|--------|
| 1        | MM      | PAID    | 25.00  |
| 2        | MM      | PAID    | 12.50  |
| 3        | TH      | PENDING | 40.00  |
| 4        | MM      | PAID    | 8.00   |

There are three main ways to flatten it into a file.

### 1. Row-based (CSV)

Store one row after another:

```text
1,MM,PAID,25.00 | 2,MM,PAID,12.50 | 3,TH,PENDING,40.00 | 4,MM,PAID,8.00
```

### 2. Column-based

Store one column after another:

```text
1,2,3,4 | MM,MM,TH,MM | PAID,PAID,PENDING,PAID | 25.00,12.50,40.00,8.00
```

### 3. Hybrid (Parquet)

First split rows into groups, called **row groups**. Then, inside each row group, store the data column by column. With a row group size of 2:

```text
┌──────────────── Row Group 0 (rows 1–2) ────────────────┐
│ 1,2 │ MM,MM │ PAID,PAID │ 25.00,12.50                  │
└────────────────────────────────────────────────────────┘
┌──────────────── Row Group 1 (rows 3–4) ────────────────┐
│ 3,4 │ TH,MM │ PENDING,PAID │ 40.00,8.00                │
└────────────────────────────────────────────────────────┘
```

Parquet is usually called a **columnar** format, but as the article points out, "hybrid" is a more precise description because it stores **chunks** of columns.

Why does this matter? Analytics queries usually do two things:

- **Projection** picks columns. It is the `SELECT` part of SQL.
- **Predicate** picks rows. It is the `WHERE` part of SQL.

A column layout is great for projection, because we can read only the columns we need. Row groups allow us to skip entire groups of rows based on predicate filters. The hybrid layout supports both, which is exactly what analytics needs.

## What Is Inside a Parquet File?

A Parquet dataset is structured like this, from biggest to smallest:

```text
Root directory
└── .parquet files
    └── Row groups
        └── Column chunks
            └── Data pages
```

Inside a single `.parquet` file, the layout looks like this:

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

- **Row group**: a horizontal slice of the table, for example 100,000 rows.
- **Column chunk**: all values of one column inside one row group.
- **Data page**: the smallest unit. It holds the actual encoded values. A column chunk is split into many pages (about 1 MB each by default), so readers can decompress or skip one page at a time.
- **Footer**: the metadata. It is written at the **end** of the file, because the writer only knows the offsets and statistics after writing the data.

A reader works in reverse. It reads the last 8 bytes, finds the footer, and then has a map of the whole file **before** reading any data.

## Why Parquet Takes Less Space

### 1. Binary types instead of text

In CSV, everything is text. The number `1234567890` takes 10 bytes as characters. In Parquet, it is stored as an `INT32` and takes only 4 bytes. A timestamp like `2026-09-14T22:00:00Z` takes 20 bytes as text, but 8 bytes as an `INT64`. And we haven't compressed anything yet.

### 2. Run-length encoding (RLE)

The article has a nice example. Imagine a column with 10,000,000 values, and every value is `0`. Instead of storing 10 million zeros, we only need two numbers: the value and how many times it repeats.

```text
Raw:   0 0 0 0 0 0 0 ... (10,000,000 times)

RLE:   (value = 0, count = 10,000,000)
```

That saves us from storing 9,999,998 numbers. In real data, values rarely repeat perfectly, but columns like `status` or `country` often have long runs, especially when the data is **sorted**.

### 3. Dictionary encoding with bit-packing

Some columns contain long strings that repeat, like country names. Storing "The Democratic Republic of Congo" (32 characters) again and again wastes a lot of space.

Dictionary encoding stores each distinct value once, and replaces the values in the column with small integers:

```text
Raw values:   MM  MM  TH  MM  MM  SG  TH  MM

Dictionary:   0 → MM
              1 → TH
              2 → SG

Indexes:      0   0   1   0   0   2   1   0
```

Those integers are **bit-packed**. With only 3 distinct values, each index needs just 2 bits, not a full 4-byte integer.

Let's do quick math for 10 million rows with 200 distinct countries, where each name is about 10 bytes on average:

```text
Plain strings:   10,000,000 × ~10 bytes  ≈ 100 MB
Dictionary:      200 × ~10 bytes         ≈   2 KB
Indexes:         10,000,000 × 8 bits     ≈  10 MB   (200 values fit in 8 bits)
```

That is about 10x smaller, before any compression.

### 4. Delta encoding

For values that grow steadily, like IDs or timestamps, storing the difference between neighbors is cheaper than storing each full value:

```text
Timestamps:  1757858400  1757858401  1757858403  1757858404

Delta:       1757858400  +1  +2  +1
```

The differences are tiny numbers, so they bit-pack into just a few bits each.

### 5. Compression on top of encoding

After encoding, each page is compressed with a codec such as **Snappy**, **GZIP**, or **ZSTD**:

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

Compression works best on data that looks similar, and a single column of the same type is exactly that. If you GZIP a CSV file, the compressor sees numbers, strings, and commas all mixed together, so it can't do as well.

### 6. Nulls cost almost nothing

Parquet doesn't store nulls as values. It stores **definition levels**, which are small numbers saying whether a value exists, and those are RLE-encoded too. The article mentions **repetition levels** as well. Together, they let Parquet store nested data like lists and structs in columns.

## Why Parquet Queries Are Fast

Let's use this query on a bigger `orders` table with 40 columns, including `order_date`:

```sql
SELECT country, SUM(amount)
FROM orders
WHERE order_date = '2026-09-01'
GROUP BY country;
```

### 1. Projection: read only the columns you need

The query uses only 3 of the 40 columns, so the query engine only reads those 3 column chunks. The other 37 are never read from disk or downloaded from S3.

```text
              id   country   status   amount   order_date   ...35 more
Row Group 0    ✗         ✓        ✗        ✓            ✓            ✗
Row Group 1    ✗         ✓        ✗        ✓            ✓            ✗
Row Group 2    ✗         ✓        ✗        ✓            ✓            ✗
```

With CSV, the engine has to read and parse every byte of every row, even if it needs only one column.

### 2. Predicate pushdown: skip row groups with min/max statistics

This is my favorite optimization feature. The footer stores the **min and max** value of every column in every row group. Before reading any data, the engine compares the `WHERE` clause with those statistics:

```text
WHERE order_date = '2026-09-01'

Row Group 0   min=2026-08-01  max=2026-08-15   → skip
Row Group 1   min=2026-08-16  max=2026-08-31   → skip
Row Group 2   min=2026-09-01  max=2026-09-14   → READ
Row Group 3   min=2026-09-15  max=2026-09-30   → skip
```

Three out of four row groups are skipped without reading them.

One important catch: this only works well when the data is **sorted or clustered** by the filter column. If dates are shuffled randomly, every row group has a min near the beginning and a max near the end, and nothing can be skipped.

The same idea works at the file level too. A dataset is often many `.parquet` files in one directory, so an engine can check each file's metadata and skip whole files.

### 3. Reading the footer first

Because all the metadata is in the footer, the engine can plan everything up front: which files, which row groups, which columns. On S3, this means a few small range requests instead of downloading whole files:

```text
1. Read last 8 bytes         → footer length
2. Read footer               → schema, offsets, min/max
3. Decide what to skip       → list of byte ranges
4. Fetch only those ranges   → decode → return result
```

### 4. Lazy evaluation in Spark

The article also points out that Spark evaluates operations lazily. Nothing runs until you actually ask for a result, so Spark can look at the whole query first and push the projection and predicates down into the Parquet reader. The data is filtered **during** the read, not after loading the full table into memory.

### 5. Parallel and vectorized reads

Row groups are independent, so different threads or machines can read them at the same time. And since a column chunk is a packed array of one type, engines like Spark, DuckDB, and Arrow can process values in batches, which is much faster for the CPU than going row by row.

## Trying It Yourself with DuckDB

You don't need Spark to see this. With [DuckDB](https://duckdb.org/), you can convert a CSV file to Parquet like this:

```sql
COPY (SELECT * FROM 'orders.csv' ORDER BY order_date)
TO 'orders.parquet' (FORMAT parquet, COMPRESSION zstd);
```

Notice the `ORDER BY`. Sorting by the column we filter on makes min/max skipping and RLE much more effective.

Then look inside the file:

```sql
-- Row groups, encodings, compression, and min/max statistics
SELECT row_group_id, path_in_schema, encodings, compression,
       stats_min, stats_max, total_compressed_size
FROM parquet_metadata('orders.parquet');

-- Schema stored in the footer
SELECT * FROM parquet_schema('orders.parquet');
```

Compare the file sizes of `orders.csv` and `orders.parquet`. You will probably be surprised.

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

Parquet is built for analytics, so it has real trade-offs:

- **Files are immutable.** You can't update a single row in place. Changing data means rewriting the file.
- **Not for point lookups or OLTP.** Fetching one full row by ID touches every column.
- **Too many small files hurt.** The article warns about this too: Parquet writers can produce many files, and for small datasets you should repartition before writing. Every file has a footer to open and read.
- **Writing is slower.** The writer buffers a whole row group in memory, builds dictionaries, and compresses the data.
- **Not human-readable.** You cannot directly view raw contents using standard terminal commands like `cat`.
- **`SELECT *` benefits less.** If you read every column, projection saves nothing, although compression still helps.

## What About Updates and Transactions?

Because Parquet files are immutable, the article mentions [Delta Lake](https://delta.io/) as the next step. Delta Lake adds a transaction log on top of Parquet files, which gives you **ACID transactions**, updates, and deletes, while the data itself is still stored as Parquet. [Apache Iceberg](https://iceberg.apache.org/) is another option that solves the same problem.

## Practical Tips

- **Sort by columns you filter on.** You get better row group skipping and better RLE compression at the same time.
- **Choose a sensible row group size.** Too small means more metadata and weaker compression. Too large means less skipping and more memory while writing.
- **Use ZSTD** if the data is read often and storage or network cost matters. Snappy is faster but produces bigger files.
- **Partition large datasets** by a column like date (for example, `orders/date=2026-09-01/part-0.parquet`), so engines can skip whole directories.
- **Avoid lots of tiny files.** Compact them regularly.
- **Use proper types.** Store dates as `DATE` or `TIMESTAMP`, not strings, so statistics and encodings work well.

## Conclusion

Parquet is small and fast for the same reason: it stores **chunks of columns** with **metadata about each chunk**. Storing columns together makes the data repetitive and similar, so RLE, dictionary encoding, and compression shrink it a lot. The metadata in the footer lets the query engine read only the columns it needs and skip row groups and files that can't match the filter.

If your workload is analytics, where you read many rows but only a few columns, Parquet is one of the best defaults you can choose. If you need frequent single-row updates, a row-based database is still the right tool. I highly recommend reading [the original article](https://towardsdatascience.com/demystifying-the-parquet-file-format-13adb0206705/) by Michael Berk too. It's a beginner-friendly introduction.
