# Usage & API

The public API lives at the module root (`github.com/go-ruby-activerecord/activerecord`).
It is **Ruby-shaped but Go-idiomatic**: `Where` / `Order` / `Joins` / `ToSQL`
mirror ActiveRecord's relation methods, while the surface follows Go conventions
— an explicit `error`, value types, no global state.

!!! success "Status: implemented"
    The library is built and importable as `github.com/go-ruby-activerecord/activerecord`,
    bound into `rbgo` as its ORM backend; see [Roadmap](roadmap.md).

## Install

```sh
go get github.com/go-ruby-activerecord/activerecord
```

## Worked example — Relation → SQL

```go
package main

import (
	"fmt"

	ar "github.com/go-ruby-activerecord/activerecord"
)

func main() {
	users := ar.NewModel("User", "users",
		ar.Column{Name: "id", Type: "integer"},
		ar.Column{Name: "name", Type: "string"},
		ar.Column{Name: "age", Type: "integer"},
		ar.Column{Name: "company_id", Type: "bigint"})
	posts := ar.NewModel("Post", "posts",
		ar.Column{Name: "id", Type: "integer"},
		ar.Column{Name: "user_id", Type: "bigint"})
	users.Register(posts).HasMany("posts", "Post")
	posts.Register(users).BelongsTo("user", "User")

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
}
```

## Validations

```go
w := ar.NewModel("Widget", "widgets", ar.Column{Name: "name", Type: "string"})
w.ValidatesPresence("name")

rec := w.Build(map[string]any{}) // no name
errs := rec.Validate()
fmt.Println(errs.FullMessages()) // ["Name can't be blank"]
```

The `Errors` object is shaped like `ActiveModel::Errors`: `FullMessages` returns
ActiveRecord's default message text in its exact order.

## Shape

```go
// NewModel declares a model with a class name, table name and columns.
func NewModel(class, table string, cols ...Column) *Model

// Relation is lazy, immutable and chainable; ToSQL renders it.
func (r *Relation) Where(cond any, args ...any) *Relation
func (r *Relation) Order(o any) *Relation
func (r *Relation) Joins(assocs ...string) *Relation
func (r *Relation) Limit(n int) *Relation
func (r *Relation) ToSQL() string

// Validations register on the model; Build + Validate run them.
func (m *Model) ValidatesPresence(attrs ...string)
func (m *Model) Build(attrs map[string]any) *Record
func (rec *Record) Validate() *Errors
func (e *Errors) FullMessages() []string
```

Aggregate and DML builders (`CountSQL` / `SumSQL` / `PluckSQL` / `ExistsSQL` /
`InsertSQL` / `UpdateAllSQL` / `DeleteAllSQL`), schema DDL (`CreateTable` /
`AddColumnSQL` / `AddIndexSQL` / `AddForeignKeySQL`), the full validator set
(`length` / `format` / `numericality` / `inclusion` / `exclusion` /
`uniqueness`), and attribute dirty tracking (`Changed` / `Changes` /
`AttributeChanged`) round out the surface.

## The Adapter (host) seam

Execution is injected. A host implements `Adapter` over its driver
(go-ruby-sqlite3 / go-ruby-pg) and the package's `Exists` / `Count` / `LoadAll`
helpers run the rendered SQL through it:

```go
type Adapter interface {
	Execute(sql string) ([]Row, error)
	ExecuteDML(sql string) (affected, lastInsertID int64, err error)
	AdapterName() string // "sqlite3" | "postgresql" | "mysql2"
}
```

`ValidatesUniqueness(attr, exists)` takes a callback the host wires to
`Exists(adapter, relation)`, so the one query-dependent validator stays behind
the seam too.

## MRI conformance

Correctness is defined by the real `activerecord` gem. A **differential oracle**
runs a wide corpus through both the gem's `Relation#to_sql` /
`errors.full_messages` / schema DDL and this library and compares the results
**byte-for-byte** — not approximated from memory. The oracle is version-gated
(`RUBY_VERSION >= "4.0"`) and skips itself where MRI is absent (the qemu arch
lanes and Windows), so the cross-platform builds still validate the library from
Ruby-free golden vectors.

## Relationship to Ruby

`go-ruby-activerecord/activerecord` is **standalone and reusable**, and is the
ORM backend bound into [go-embedded-ruby](https://github.com/go-embedded-ruby/ruby)
by `rbgo`. The dependency runs the other way: this library has no dependency on
the Ruby runtime.
