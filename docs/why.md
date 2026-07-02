# Why pure Go

`go-ruby-activerecord/activerecord` reimplements the core of Rails' ActiveRecord
ORM in **pure Go, with cgo disabled**. The slice of ActiveRecord it covers is
**deterministic and database-independent**: building the SQL string for a
relation (`where` / `order` / `joins` / aggregates), the migration DDL, the
association join geometry, and the validation logic plus message text is a pure
function of the model and relation description — no live connection, no
evaluation of arbitrary Ruby. That is exactly the part that can — and should —
live as a standalone Go library, separate from both the interpreter and the
database.

## What it is — and isn't

Rendering the SQL, the DDL and the validation errors needs **no interpreter and
no live database**, so it lives here as pure Go. Actually running the statements,
connection pooling, transaction *execution*, the full `before/after` callback
chain and STI edge cases are the host's job. This library renders the SQL and
exposes an [`Adapter`](api.md#the-adapter-host-seam) the host implements over its
driver ([go-ruby-sqlite3](https://github.com/go-ruby-sqlite3/sqlite3) /
[go-ruby-pg](https://github.com/go-ruby-pg/pg)).

## Standalone, reusable, and the rbgo backend

This library is the ORM backend for
[go-embedded-ruby](https://github.com/go-embedded-ruby/ruby)'s `rbgo`, bound as a
native module — the same pattern as the other go-ruby-\* libraries — while also
being a **standalone, reusable** module. So that:

- any Go program can import `github.com/go-ruby-activerecord/activerecord`
  directly, with no Ruby runtime;
- the dependency runs the *other* way — `rbgo` binds this module as its ORM
  backend rather than this module depending on the interpreter;
- it is a sibling of [go-ruby-sequel](https://github.com/go-ruby-sequel/sequel),
  whose per-dialect approach it shares;
- the behaviour is pinned by a **differential oracle** against the real
  `activerecord` gem, independent of any one consumer.

## Why pure Go matters here

Because the library is CGO-free and dependency-free, it:

- cross-compiles to every Go target with no C toolchain, and links into a single
  static binary;
- has **no dependency on the Ruby runtime or a database driver** — execution is
  injected through the `Adapter` seam, so the SQL-generation core builds and runs
  everywhere;
- can be differentially tested against the `activerecord` gem wherever MRI is on
  `PATH`, while the cross-arch and Windows lanes (where no MRI is present) still
  validate the library from Ruby-free golden vectors.

See [Usage & API](api.md) for the surface and [Roadmap](roadmap.md) for what is
in scope.
