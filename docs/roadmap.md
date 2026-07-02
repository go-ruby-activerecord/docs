# Roadmap

`go-ruby-activerecord/activerecord` is grown **test-first**, each capability
differential-tested against the real `activerecord` gem rather than built in
isolation. The deterministic, database-independent core of ActiveRecord —
extracted as pure Go — is **complete**.

| Stage | What | Status |
| --- | --- | --- |
| Relation & ToSQL | A lazy, immutable, chainable `Relation` — `Where` (hash / `?`- and `:name`-placeholders / IN / BETWEEN / IS NULL), `Not`, `Or`, `Select`, `Order`, `Limit`/`Offset`, `Group`/`Having`, `Distinct`, `Merge`, named scopes and finders — rendered by `ToSQL()`, plus aggregate and DML string builders. | **Done** |
| Associations & joins | `belongs_to` / `has_many` / `has_one` / `has_and_belongs_to_many` and `:through`, emitting ActiveRecord's exact `INNER` / `LEFT OUTER JOIN … ON` geometry, including HABTM join tables and the two-hop through join with source-reflection direction. | **Done** |
| Schema DDL | `CreateTable` (with `References` / `Timestamps` / `NoPrimaryKey`), `AddColumnSQL`, `AddIndexSQL` (unique, default `index_<t>_on_<cols>` name), `AddForeignKeySQL`, and a per-dialect column-type map. | **Done** |
| Validations & Errors | `presence` / `length` / `format` / `numericality` / `inclusion` / `exclusion` / `uniqueness` producing an `Errors` object shaped like `ActiveModel::Errors`, with ActiveRecord's default message text and `full_messages` ordering. | **Done** |
| Attributes & dirty tracking | Attribute readers / writers, per-column type casting, and dirty tracking (`Changed` / `Changes` / `AttributeChanged`) mirroring `ActiveModel::Dirty`. | **Done** |
| Dialects (SQLite / PG / MySQL) | SQLite, PostgreSQL and MySQL identifier quoting and value literalization, selectable per model. | **Done** |
| Differential oracle & coverage | Generated SQL, `errors.full_messages` and the schema DDL compared to the real `activerecord` gem byte-for-byte; 100% coverage, gofmt + go vet clean, green across all six 64-bit Go arches and three OSes. | **Done** |

## Documented out-of-scope boundaries

These are **deliberate** host seams, recorded so the module's surface is
unambiguous:

- **Statement execution.** The library renders SQL; running it, connection
  pooling and transaction *execution* are injected through the `Adapter` seam —
  that is why `rbgo` binds this module rather than the reverse.
- **Callbacks, STI edge cases, eager loading.** The full `before/after` callback
  chain execution (registration is a host concern; bodies run in `rbgo`), STI
  edge cases, and eager-loading materialization are the host's job. `includes`
  query *planning* lives here; the multi-query fetch does not.
- **Reference is the real gem (MRI).** Byte-for-byte conformance targets the
  `activerecord` gem's behaviour, matched to the reference used by the
  differential oracle.
- **Standalone & reusable.** The module has no dependency on the Ruby runtime;
  the dependency runs the other way.

See [Usage & API](api.md) for the surface and [Why pure Go](why.md) for the
deterministic / execution split.
