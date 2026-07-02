# Contributing

Contributions are welcome. `go-ruby-activerecord/activerecord` is built to a
small set of non-negotiable rules — they are what keep it pure-Go, correct, and
MRI-compatible. Please read these before opening a pull request.

## Hard rules

- **Build from source — no vendoring.** Everything compiles from source. Being
  able to compile from source is a guarantee of independence.
- **100% test coverage target, enforced in CI.** New code ships with tests, and
  coverage is a CI gate. Fill the error branches, not just the happy path.
- **All GitHub content in English.** Issues, pull requests, commits, comments,
  and discussions are English-only.
- **Differential testing against MRI.** Correctness is defined by the real
  `activerecord` gem. Generated SQL, `errors.full_messages` and schema DDL are
  run through both the gem and this library and compared byte-for-byte — not
  approximated from memory.
- **Pure Go, cgo disabled.** The whole point is a single static binary with no C
  toolchain. Code must build with `CGO_ENABLED=0`. Statement execution stays
  behind the `Adapter` seam; if a feature seems to need a live database, it needs
  a pure-Go SQL-generation path plus a host seam instead.
- **A reusable library, not the interpreter or the database.** This module
  implements the deterministic core (SQL generation, associations, validations).
  Anything that needs a live Ruby binding or a live connection belongs in the
  consumer, behind the `Adapter`, not here.

## Workflow

1. Pick or open an issue describing the change.
2. Work test-first: add the differential / unit tests, then make them pass.
3. Run the full suite with coverage and confirm the gate is green:

    ```sh
    GOWORK=off go test -race -coverprofile=cover.out ./...
    GOWORK=off go tool cover -func=cover.out | tail -1   # 100.0%
    ```

4. Open a PR in English, referencing the issue.

## Where things live

The library is in
[`github.com/go-ruby-activerecord/activerecord`](https://github.com/go-ruby-activerecord/activerecord).
This documentation site is in
[`github.com/go-ruby-activerecord/docs`](https://github.com/go-ruby-activerecord/docs).
Start from the [Usage & API](api.md) page and the [Roadmap](roadmap.md) to find
the right place for your change.
