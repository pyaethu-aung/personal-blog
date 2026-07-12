+++
title = "Why I Use sqlc Instead of GORM in My Image Server Project with Go"
date = '2026-07-12T22:00:00+07:00'
draft = false
slug = "sqlc-go"
tags = ["go", "sqlc", "gorm", "postgresql", "database"]
+++

[မြန်မာဘာသာဖြင့် ဖတ်ရှုရန်]({{< relref path="2026071201/index.my.md" lang="my" >}})

## Introduction

Recently, I have been building [image-server](https://github.com/pyaethu-aung/image-server), an image upload and transformation server in Go. For the database layer, I had to choose between two popular options: an ORM like [GORM](https://gorm.io/), or a code generator like [sqlc](https://sqlc.dev/).

I chose sqlc. In this post, I will share what sqlc is, how it works, its pros and cons, a quick comparison with GORM, and how I configured it in my project, including the git hooks that keep the generated code safe.

## What is sqlc?

sqlc is not an ORM. You write plain SQL in `.sql` files, and sqlc generates type-safe Go code from them. There is no query builder and no magic at runtime. Everything happens at build time, before your code even compiles.

For example, this is a query file in my project:

```sql
-- name: GetImage :one
SELECT * FROM images
WHERE id = $1;

-- name: CreateImage :one
INSERT INTO images (
    original_filename, content_hash, mime_type,
    width, height, size_bytes, storage_key
) VALUES (
    $1, $2, $3, $4, $5, $6, $7
)
RETURNING *;
```

After running `sqlc generate` in terminal, I get a Go function like this:

```go
func (q *Queries) GetImage(ctx context.Context, id uuid.UUID) (Image, error)
```

The `Image` struct is also generated automatically, with the correct Go types for every column (`uuid.UUID`, `int32`, `int64`, and so on). I don't need to write this code by hand.

## Where Does sqlc Get the Table Structure?

sqlc does **not** connect to a live database. In `sqlc.yaml`, I point it at migration files:

```yaml
version: "2"
sql:
  - engine: "postgresql"
    queries: "internal/db/queries"
    schema: "migrations"
    gen:
      go:
        package: "db"
        out: "internal/db"
        sql_package: "pgx/v5"
```

sqlc reads every migration in the `migrations/` folder, replays them in order like a fresh database, and builds the schema in memory. It is smart enough to skip `.down.sql` rollback files and only apply the `.up.sql` files. Then it type-checks every query against that schema.

So if I rename a column in a migration but forget to update a query, code generation fails. I find out at build time, not in production.

## Pros of sqlc

- **You write real SQL.** No new DSL to learn.
- **Compile-time safety.** Wrong types, wrong column names, and wrong parameter counts are all caught before the code runs.
- **No hidden queries.** The SQL that hits the database is exactly the SQL you wrote. No surprise **N+1** queries.
- **No runtime overhead.** The generated code is just `QueryRow` and `Scan`. No reflection.
- **SQL injection safe by construction.**" Every query is parameterized (`$1`, `$2`), so string concatenation never happens.

## Cons of sqlc

- **You must write SQL.** If your team wants to avoid SQL completely, this is a wall, not a feature.
- **Dynamic queries are awkward.** Queries are static strings, so building a search endpoint with five optional filters is painful compared to chaining `.Where()` calls.
- **No relations or eager loading.** If you want joined data, you write the `JOIN` yourself.
- **Extra build step.** Every query change needs a re-generate, and the output must be committed. (Git hooks can solve this, see below.)
- **Migrations are your job.** sqlc does not manage schema changes like `AutoMigrate()` function in GORM; you need a separate migration tool.

## Quick Comparison with GORM

| | sqlc | GORM |
|---|---|---|
| What you write | Plain SQL | Go structs + method chains |
| Type safety | Compile time | Runtime (reflection) |
| Query visibility | Exactly what you wrote | Built for you behind the scenes |
| Performance | No reflection, no overhead | Reflection + possible N+1 queries |
| Dynamic queries | Clunky | Easy |
| Learning curve | Know SQL, tiny Go API | Learn GORM's conventions |
| Migrations | Separate tool | `AutoMigrate` (risky for production) |

Neither one is "better" in every case. GORM shines when you have many entities with relations and lots of dynamic filtering. sqlc shines when you want full control, predictable performance, and queries you can read.

## Why I Chose sqlc for My Image Server

[image-server](https://github.com/pyaethu-aung/image-server) has one security rule that is non-negotiable: **parameterized queries only**. There is simply no way to build a query by string concatenation, so SQL injection is not something I can introduce by accident.

On top of that, my queries are simple CRUD: create an image record, get it by ID, look it up by content hash for deduplication, and delete it. There is no dynamic filtering at all, so sqlc's biggest weakness never comes up, and its biggest strength (exact SQL with compile-time checking) is all upside.

## Keeping Generated Code in Sync with Git Hooks

Generated code has one classic failure mode: someone edits a query, forgets to re-generate, and commits stale code. I close that gap with git hooks, committed in `.githooks/` and activated once per clone:

```bash
git config core.hooksPath .githooks
```

### 1. `pre-commit`: drift check

The `pre-commit` hook re-runs the generator and fails the commit if the output differs from what is staged:

```bash
if [[ -f sqlc.yaml ]]; then
  make -s sqlc-gen > /dev/null
  if ! git diff --exit-code -- internal/db > /dev/null 2>&1; then
    echo "❌ internal/db is out of sync with the SQL queries."
    echo "   Run 'make sqlc-gen', stage the result, and recommit."
    exit 1
  fi
fi
```

So it is impossible to commit a query change without its generated code. The same hook does the identical check for my OpenAPI-generated code.

### 2. `pre-push`: quality gates

The `pre-push` hook runs the heavier gates before anything leaves my machine: `golangci-lint`, a 90% test-coverage gate, API tests that validate every endpoint against the OpenAPI spec, and full-stack e2e tests against the real Docker container. CI re-runs the first three gates repo-side, so even `--no-verify` cannot sneak bad code onto the main branch.

The result: schema, queries, and Go types can never silently drift apart.

## Conclusion

If your project has complex relations and lots of dynamic queries, GORM is still a reasonable choice. But if you are comfortable with SQL and you value compile-time safety, predictable performance, and knowing exactly what runs against your database, I highly recommend giving sqlc a try. For my [image-server](https://github.com/pyaethu-aung/image-server) project, it has been the right call, and with the git hooks in place, the generated code takes care of itself.
