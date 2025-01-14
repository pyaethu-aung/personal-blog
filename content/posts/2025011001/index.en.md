+++
title = "Cursor Pagination"
date = '2025-01-12T23:07:00+07:00'
draft = true
lang = "en"
slug = "cursor-pagination"
tags = ["postgres", "mongodb", "sql", "nosql"]
+++

Developers who work on mobile and web apps, those who have written CRUD APIs, and who have worked with third-party APIs are likely to be very familiar with pagination. If you’re building an API, you might add query parameters like `limit`, and `ski`p to the URL. As part of the validation, you might set a maximum value for limit, and so on.

For mobile developers, [infinite scroll](https://en.wiktionary.org/wiki/infinite_scroll) often require triggering the next API call when the user scrolls to a certain point. Since these concepts are likely well-known to most of you, I’ll skip straight to the main point.

## Offset Pagination
The reason of discussing offset pagination before cursor pagination is that it’s one of the most widely used methods. It works in a simple way: by providing query parameters like `?limit=20&skip=40`, it skips the first 40 records in a sorted dataset and returns the subsequent 20 records.

It can be written in SQL like below:
```sql
SELECT *
FROM records
OFFSET 20
LIMIT 20;
```

It's similar in MongoDB too.
```javascript
db.records.find({}, { limit: 20, skip: 20 })
```

### Drawbacks of Offset Pagination
Offset pagination has two main drawbacks:

#### 1. Performance Issue on Large Dataset
The first drawback is performance issue. When the dataset becomes large, using `OFFSET` can cause significant problems as it requires scanning the entire table to retrieve the data. This issue worsens with complex queries, potentially impacting the performance of the entire database. In a microservices architecture, such queries can even affect the overall service performance.

#### 2. Inconsistent Data When Insert/Delete
The second drawback is the inability to provide consistent data. This issue may not be noticeable in systems with infrequent changes. However, for systems like social media apps, B2C, or C2C marketplace apps, the inability to deliver consistent data can become a significant problem.

For example, imagine a database table containing 100 records with IDs ranging from `1` to `100`, and you want to fetch data in descending order. On the first request,you would receive 20 records with IDs from `100` to `81` by using `OFFSET 0 LIMIT 20`. On the second request, using `OFFSET 20 LIMIT 20`, you would get the next 20 records with IDs from `80` to `61`. So far, everything seems to be working fine.

Now, imagine that before the second request, a new record with ID `101` is added to the table. This is where the problem begins. Since the second query uses `OFFSET 20 LIMIT 20`, it skips 20 records starting from ID `101`, which results in skipping records from IDs `101` to `82` and returning records from IDs `81` to `62`. As a result, the record with ID `81` gets included again in the second set, causing duplicates.

In scenarios like infinite scroll, this duplication can lead to a poor user experience where the same record is displayed twice. In small teams with only one QA tester, this issue can sometimes go unnoticed during the testing phase.

The same issue arises if a record is deleted during pagination. For example, if the record with ID `100` is deleted before the second query, the result changes. Using `OFFSET 20 LIMIT 20` will now return records from ID `79` to `60`, completely skipping ID `80`. This issue can occur with both hard deletes and soft deletes that utilize a deleted_at column.

## Cursor Pagination
Cursor pagination is different from offset pagination. It uses identifiers like [ULID](https://github.com/ulid/spec) or timestamps such as `created_at` to paginate through data.

It can be written in SQL like below:
```sql
SELECT *
FROM records
OFFSET 20
LIMIT 2
```

It's similar in MongoDB too.
```javascript
db.records.find({}, { limit: 20, skip: 20 })
```
