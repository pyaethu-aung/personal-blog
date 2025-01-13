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
