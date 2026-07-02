# go-ruby-activerecord documentation

**Rails' ActiveRecord ORM core in pure Go — SQL generation, associations and validations, no cgo.**

`go-ruby-activerecord/activerecord` is a faithful, pure-Go (zero cgo)
reimplementation of the deterministic core of Rails' ActiveRecord ORM, matching
the real `activerecord` gem byte-for-byte. The module path is
`github.com/go-ruby-activerecord/activerecord`.

It turns a model plus a relation description into SQL and runs validations,
exactly as MRI's `activerecord` gem does — while the one thing that genuinely
needs a database, **executing** the SQL, is an injected **Adapter** host seam
(wired to [go-ruby-sqlite3](https://github.com/go-ruby-sqlite3/sqlite3) /
[go-ruby-pg](https://github.com/go-ruby-pg/pg)). So the module stays 100% Ruby-
and CGO-free, and is both the ORM backend bound into
[go-embedded-ruby](https://github.com/go-embedded-ruby/ruby) by `rbgo` — a
sibling of [go-ruby-sequel](https://github.com/go-ruby-sequel/sequel) whose
dialect approach it shares — and a **standalone, reusable** module.

!!! success "Status: complete — MRI byte-exact"
    Faithful port of ActiveRecord's SQL generation and validations: a lazy, chainable **`Relation`** rendered by **`ToSQL`**, aggregate and DML builders, **associations** (`belongs_to` / `has_many` / `has_one` / HABTM / `:through`) emitting ActiveRecord's exact join geometry, **schema DDL**, **validations** producing an `ActiveModel::Errors`-shaped `Errors`, **attributes** with dirty tracking, and **three dialects** (SQLite / PostgreSQL / MySQL). Validated by a **differential oracle** against the real `activerecord` gem — generated SQL, `errors.full_messages` and DDL compared byte-for-byte — at 100% coverage, `gofmt` + `go vet` clean, CI green across the six 64-bit Go targets and three OSes.

## Quick taste

```go
rel := users.
    Where(map[string]any{"age": &ar.Range{Begin: 18, End: 30}}).
    Joins("posts").
    Order(map[string]any{"name": "asc"}).
    Limit(10)

fmt.Println(rel.ToSQL())
// SELECT "users".* FROM "users"
//   INNER JOIN "posts" ON "posts"."user_id" = "users"."id"
//   WHERE "users"."age" BETWEEN 18 AND 30
//   ORDER BY "users"."name" ASC LIMIT 10
```

## Repositories

| Repo | What it is |
| --- | --- |
| [`activerecord`](https://github.com/go-ruby-activerecord/activerecord) | the library — Rails' ActiveRecord ORM core in pure Go |
| [`docs`](https://github.com/go-ruby-activerecord/docs) | this documentation site (MkDocs Material, versioned with mike) |
| [`go-ruby-activerecord.github.io`](https://github.com/go-ruby-activerecord/go-ruby-activerecord.github.io) | the organization landing page (Hugo) |
| [`brand`](https://github.com/go-ruby-activerecord/brand) | logo and brand assets |

## Principles

- **Pure Go, `CGO_ENABLED=0`** — trivial cross-compilation, a single static
  binary, no C toolchain; SQL execution is injected through an `Adapter` seam.
- **MRI byte-exact.** Generated SQL, `errors.full_messages` and schema DDL match
  the real `activerecord` gem exactly, validated by a differential oracle against
  ActiveRecord's own `Relation#to_sql`.
- **Standalone & reusable.** The ORM backend bound into `rbgo`, and importable by
  any Go program; the dependency runs that way, not the reverse.
- **100% test coverage** is the target, enforced as a CI gate, across 6 arches
  and 3 OSes.

## Where to go next

- [Why pure Go](why.md) — why this slice of ActiveRecord is deterministic enough
  to live as a standalone, database-independent Go library.
- [Usage & API](api.md) — the public surface and worked examples.
- [Roadmap](roadmap.md) — what is done and what is downstream by design.

Source lives at [github.com/go-ruby-activerecord/activerecord](https://github.com/go-ruby-activerecord/activerecord).
