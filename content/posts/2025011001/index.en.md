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
WHERE created_at < 1736533304
LIMIT 20;
```

It's similar in MongoDB too.
```javascript
db.records.find(
  { created_at: { $lt: 1736533304 } }
).limit(20);
```

### Pros of Cursor Pagination
Since condition-based query can be used, the issues encountered in [offset pagination](#offset-pagination) are effectively resolved.

#### 1. Efficient Queries and Index Utilization
Since `OFFSET` is no longer used, there is no need to scan the entire table or collection, resulting in improved query performance.

With condition-based skipping, it becomes possible to create indexes specifically for pagination. Whether it’s SQL or NoSQL, indexes cannot be created for `OFFSET`, but they can be utilized effectively in this approach.

#### 2. Consistent Data When Insert/Delete
With cursor pagination, issues like missing or duplicated data caused by frequent inserts or deletes in offset pagination are resolved. This approach ensures consistent data retrieval, regardless of how frequently data is inserted or deleted, eliminating the possibility of data loss.

However, in systems where data additions or deletions occur rarely, it might not be necessary to consider these issues as a priority.

### Drawbacks of Cursor Pagination
Cursor pagination is not a silver bullet, and it comes with its own limitations.

#### 1. Complexity
Cursor pagination is more complex to implement compared to [offset pagination](#offset-pagination). It requires additional calculations and considerations. For instance, if you prefer not to use an `AUTO_INCREMENT` ID or cannot rely on the `created_at` column due to some reasons, you might need to adopt alternatives like UUID v7. Furthermore, if the cursor provided by the client is invalid and no data is returned from database, you’ll need to handle the business logic to decide which data to send back.

#### 2. Inflexibility
Cursor pagination works well when navigating sequentially. However, it’s not suited for jumping directly between pages (e.g., from page 1 to page 5). Achieving this requires additional workarounds, and even then, the solution might not be efficient.

Another drawback is that reverse navigation can be challenging. For instance, if you’re on page 5 and want to go back to page 4, it’s not straightforward. To handle this, you’d need to store the data for previous pages locally, such as in local storage.

These issues are less significant in scenarios like infinite scroll, where sequential loading of data is the norm.

#### 3. Depends on Sort and Order By
This limitation ties into the [inflexibility](#2-inflexibility) issue. For example, if you’re on page 5 and decide to sort by a different column, such as the name column, or switch from ascending to descending order, it’s not feasible with cursor pagination. Even if you manage to implement such functionality, it won’t be efficient. While there are workarounds to achieve these tasks, they often come with compromises, which is why it’s important to highlight this upfront.

#### 4. Misc
Another consideration, though rare, is columns used for sorting must be unique. While this isn’t an issue for fields like ID, it can raise a problem for columns like `created_at`, where duplicate values are possible, especially down to the millisecond level.

## Conclusion
