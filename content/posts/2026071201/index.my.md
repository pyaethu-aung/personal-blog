+++
title = "ဘာလို့ GORM အစား sqlc လဲ"
date = '2026-07-12T22:00:00+07:00'
draft = false
slug = "sqlc-go"
tags = ["go", "sqlc", "gorm", "postgresql", "database"]
+++

[Read this article in English]({{< relref path="2026071201/index.en.md" lang="en" >}})

## နိဒါန်း

ဒီရက်ပိုင်း [image-server](https://github.com/pyaethu-aung/image-server) ဆိုတဲ့ image upload နဲ့ transformation server တခုကို Go နဲ့ ရေးနေတာရှိပါတယ်။ [GORM](https://gorm.io/) လို ORM ကို သုံးမလား၊ ဒါမှမဟုတ် [sqlc](https://sqlc.dev/) လို code generator ကို သုံးမလား ဆိုပြီး ရွေး ချယ်စရာ ၂ ခုရှိလာတာမှာ ကျွန်တော်ကတော့ sqlc ကို ရွေးဖြစ်လိုက်ပါတယ်။

ဒီ post မှာတော့ sqlc ဆိုတာ ဘာလဲ၊ ဘယ်လိုအလုပ်လုပ်လဲ၊ အားနည်းချက် အားသာချက်တွေက ဘာတွေလဲ၊ GORM နဲ့ ဘာကွာလဲ၊ ပြီးတော့ git hooks တွေအပါအဝင် ကျွန်တော့ရဲ့ project ထဲမှာ ဘယ်လို configure လုပ်ထားလဲ ဆိုတာတွေကို sharing လုပ်ချင်ပါတယ်။

## sqlc ဆိုတာ ဘာလဲ?

sqlc က ORM မဟုတ်ပါဘူး။ sqlc က ရိုးရိုး SQL file တွေကနေ type-safe ဖြစ်တဲ့ Go code ကို generate လုပ်ပေးတာပါ။ Query builder လည်း မရှိသလို runtime မှာမှ ပေါ်လာတဲ့ magic code တွေလည်း မရှိပါဘူး။ အားလုံးက build time မှာပဲ generate လုပ်သွားတာပါ။

ဒါက ကျွန်တော့် project ထဲက SQL query file တခုပါ။

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

`sqlc generate` command ကို run လိုက်တာနဲ့ အောက်ကလို Go function တခုကို generate လုပ်ပေးသွားပါလိမ့်မယ်။

```go
func (q *Queries) GetImage(ctx context.Context, id uuid.UUID) (Image, error)
```

`Image` struct ကိုလည်း column တိုင်းအတွက် Go type တွေ (`uuid.UUID`, `int32`, `int64`) နဲ့  generate လုပ်ပေးပါတယ်။ ကိုယ်တိုင် လုံးဝရေးစရာမလိုပါဘူး။

## sqlc က Table Structure ကို ဘယ်ကရမှာလဲ?

`sqlc.yaml` file ထဲမှာ ကိုယ့်ရဲ့ database migration file တွေရှိတဲ့ နေရာကို ညွှန်ပေးလိုက်ရုံပါပဲ။

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

`migrations` folder ထဲက SQL file တွေအားလုံးကို ဖတ်၊ `.down.sql` ဆိုတဲ့ rollback file တွေကို ကျော်ပြီး `.up.sql` file တွေကိုပဲ ယူ၊ database အသစ်မှာ run သလို အစဉ်လိုက် replay ပြန်လုပ်ပြီး schema ကို memory ထဲမှာ ဆောက်ပါတယ်။ ပြီးမှ query တိုင်းကို အဲ့ဒီ schema နဲ့ တိုက်စစ်တာပါ။

ဒါကြောင့် migration ထဲမှာ column နာမည်ပြောင်းပြီး query ကို ပြင်ဖို့ မေ့သွားခဲ့တာမျိုးဆိုရင် code generation က fail ဖြစ်ပါလိမ့်မယ်။ Production ကျမှ မဟုတ်ဘဲ build time မှာတင် သိရမှာပါ။

## sqlc ရဲ့ အားသာချက်တွေအနေနဲ့

- **SQL ပဲ ရေးဖို့လိုပါတော့မယ်။** DSL အသစ် တခုကို ထပ်လေ့လာနေစရာမလိုတော့ပါဘူး။
- **Compile-time safety ဖြစ်မယ်။** Data type မှားတာ၊ column နာမည်မှားတာ၊ parameter အရေအတွက် မှားတာတွေကို compile time မှာတင် သိပါမယ်။
- **Hidden query တွေ မရှိတော့ပါဘူး။** Database မှာ run သွားတဲ့ SQL က ကိုယ်ရေးထားခဲ့တဲ့ SQL အတိုင်း အတိအကျပါပဲ။ မမျှော်မှန်းထားတဲ့ N+1 query တွေ မရှိနိုင်တော့ပါဘူး။
- **Runtime overhead မရှိတော့ပါဘူး။** Generated code က `QueryRow` နဲ့ `Scan` ပဲ ပါပါတော့မယ်။ No reflection ပါ။
- **SQL injection မရှိတော့ပါဘူး။** Query တိုင်းက parameterized (`$1`, `$2`) ဖြစ်နေတဲ့အတွက် string concatenation မရှိနိုင်တော့ပါဘူး။

## sqlc ရဲ့ အားနည်းချက်တွေ

- **SQL ကို ကိုယ်တိုင် ရေးရမယ်။** SQL နဲ့ လုံးဝမရင်းနီးပဲ ORM သုံးနေကျ team အတွက်ဆိုရင်တော့ အဆင်မပြေပါဘူး။
- **Dynamic query တွေအတွက် အဆင်မပြေနိုင်ပါဘူး။** Query တွေက static string တွေ ဖြစ်တဲ့အတွက် optional filter ငါးခုလောက်ပါတဲ့ search endpoint မျိုး ဆောက်ရရင် `.Where()` တွေ ဆက်သုံးလို့ရတဲ့ GORM နဲ့ယှဉ်ဖို့ မဖြစ်နိုင်ပါဘူး။
- **Relation တွေ၊ eager loading တွေ သုံးလို့မရပါဘူး။** Join လုပ်ထားတဲ့ data လိုချင်ရင် `JOIN` statement ကို ကိုယ်တိုင် ရေးဖို့လိုပါမယ်။
- **Build step တခု ပိုလာပါမယ်။** Query ပြောင်းတိုင်း `generate` command ပြန် run ရပြီးတော့ ထွက်လာတဲ့ generated code ကိုပါ commit လုပ်ရပါမယ်။ (ဒါကို git hooks နဲ့ ဖြေရှင်းလို့ရပါတယ်။ နောက်မှာဆက်ရှင်းပြထားပါတယ်။)
- **ကိုယ်တိုင် migrate လုပ်ရပါမယ်။** sqlc က schema ပြောင်းတာတွေအတွက် GORM ရဲ့ `AutoMigrate()` လိုမျိုး support မပေးထားပါဘူး။

## GORM နဲ့ ယှဉ်ကြည့်လို့ရအောင် table လေး လုပ်ပေးထားပါတယ်

| | sqlc | GORM |
|---|---|---|
| What you write | Plain SQL | Go structs + method chains |
| Type safety | Compile time | Runtime (reflection) |
| Query visibility | Exactly what you wrote | Built for you behind the scenes |
| Performance | No reflection, no overhead | Reflection + possible N+1 queries |
| Dynamic queries | Clunky | Easy |
| Learning curve | Know SQL, tiny Go API | Learn GORM's conventions |
| Migrations | Separate tool | `AutoMigrate` (risky for production) |

Relation တွေများပြီး dynamic filtering အများကြီးလိုတဲ့ project လိုမျိုးမှာ GORM က ပိုအဆင်ပြေပါတယ်။ Query တွေကို အတိအကျရေးချင်ပြီး performance ကို ကြိုတွက်ထားချင်တယ်ဆိုရင်တော့ sqlc က ပိုသင့်တော်ပါတယ်။

## Image Server အတွက် sqlc ကို ဘာလို့ရွေးခဲ့တာလဲ?

ကျွန်တော့် [image-server](https://github.com/pyaethu-aung/image-server) project မှာ non-negotiable security rule တခု ရှိပါတယ်။ အဲ့ဒါက **parameterized queries only** ပါ။ sqlc ကို သုံးလိုက်တာနဲ့ string concatenation နဲ့ query ဆောက်လို့ လုံးဝမရတော့တဲ့အတွက် SQL injection ဖြစ်လာစရာမရှိတော့ပါဘူး။

နောက်တခုက ကျွန်တော့် query တွေက ရိုးရိုး CRUD တွေပဲ ဖြစ်နေတာလည်း ပါပါတယ်။ Image record အသစ်ဆောက်တာ၊ ID နဲ့ ပြန်ရှာတာ၊ deduplication အတွက် content hash နဲ့ ရှာတာ၊ ဖျက်တာတွေပါပဲ။ Dynamic filtering လုံးဝမရှိတဲ့အတွက် sqlc ရဲ့ အားနည်းချက်ထက် သူ့ရဲ့ အကြီးမားဆုံးအားသာချက် (compile-time checking နဲ့ exact SQL) ရနေတာပါ။

## Generated Code တွေ Sync ဖြစ်နေအောင် Git Hooks နဲ့ ဘယ်လိုထိန်းထားလဲ?

Generated code တွေမှာ အဖြစ်များတဲ့ပြဿနာတခု ရှိပါတယ်။ တယောက်ယောက်က query ကို ပြင်ပြီး `generate` command ပြန် run ဖို့ မေ့သွားရင် generated code အဟောင်းက commit ထဲ ပါသွားတတ်ပါတယ်။ အဲ့ဒီပြဿနာကို `.githooks/` folder ထဲမှာ git hooks တွေနဲ့ ပိတ်ထားပါတယ်။ Project ကို clone လုပ်ပြီးရင် အောက်က command ကို terminal မှာ တခါ run ပေးဖို့လိုပါမယ်။ တခါပဲ လိုတာပါ။

```bash
git config core.hooksPath .githooks
```

### 1. `pre-commit`: drift check

`pre-commit` hook က generator ကို ပြန် run ပြီး ထွက်လာတဲ့ generated code က staged code နဲ့ မတူရင် commit ကို fail လိုက်ပါလိမ့်မယ်။

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

ဒါကြောင့် query ကိုပြင်ထားပြီး updated generated code မပါဘဲ commit လုပ်လို့ လုံးဝမရပါဘူး။ OpenAPI ကနေ generate လုပ်ထားတဲ့ code အတွက်လည်း ဒီ hook ထဲမှာ အတူတူပဲစစ်ထားပါတယ်။

### 2. `pre-push`: quality gates

pre-push hook ကတော့ ပိုကြီးတဲ့ gate တွေကို push မလုပ်ခင် အရင် run ပါတယ်။ `golangci-lint`၊ 90% test-coverage gate၊ endpoint တိုင်းကို OpenAPI spec နဲ့ တိုက်စစ်ထားတဲ့ API test တွေ၊ ပြီးတော့ Docker container နဲ့ run တဲ့ full-stack e2e test တွေပါပါတယ်။ ပထမ gate သုံးခုကို CI ကလည်း repo ဘက်မှာ ပြန် run တဲ့အတွက် `--no-verify` option နဲ့ဆိုရင်တောင် bad code တွေ main branch ပေါ် ရောက်မသွားနိုင်တော့ပါဘူး။

ရလဒ်ကတော့ schema၊ query တွေနဲ့ Go type တွေက အမြဲတမ်း sync ဖြစ်နေမှာပါ။

## နိဂုံး

ကိုယ့်ရဲ့ project မှာက complex relation တွေနဲ့ dynamic query တွေ အများကြီး ရှိတယ်ဆိုရင် GORM ကိုသာရွေးပါ။ ဒါပေမယ့် SQL ရေးရတာ ပိုအဆင်ပြေမယ်၊ compile-time safety ဖြစ်မယ်၊ ကြိုတွက်ထားလို့ရတဲ့ query performance လိုချင်တယ်၊ database ပေါ်မှာ ဘာ SQL run မလဲဆိုတာ အတိအကျကြိုရေးထားချင်တယ်ဆိုရင်တော့ sqlc ကို စမ်းသုံးကြည့်ဖို့ recommend လုပ်ချင်ပါတယ်။
