# Performance

`go-ruby-activerecord/activerecord` is the pure-Go library that
[`rbgo`](https://github.com/go-embedded-ruby/ruby) binds as its ORM backend. This
page records the **methodology** for a comparative benchmark of that module
against the reference Ruby runtimes, part of the ecosystem-wide per-module parity
suite.

## What is measured

The **same** Ruby script — building a representative set of ActiveRecord
relations, validations and schema DDL and materializing their SQL via
`Relation#to_sql` — is run under every runtime. `rbgo`'s number reflects **this
pure-Go library doing the work**; every other column is that interpreter's own
`activerecord` gem. So the comparison is the **Ruby-visible operation**,
apples-to-apples across interpreters. The script prints a deterministic checksum
and its output is checked **byte-identical to MRI** before timing.

- **Method:** best-of-5 wall time (best, not mean, to suppress scheduler noise);
  single-shot processes, no warm-up beyond the script's own loop.
- **Runtimes:** `ruby 4.0.x +PRISM` (MRI, the oracle) and `ruby --yjit`;
  `jruby` (OpenJDK); `truffleruby` (GraalVM CE Native).
- The benchmark script and harness live in rbgo's repo under
  [`bench/modules/`](https://github.com/go-embedded-ruby/ruby/tree/main/bench/modules).
  Reproduce:
  `RBGO=./rbgo TRUFFLE=truffleruby bash bench/modules/run.sh 5`.

## Result

!!! note "Measurement pending"
    Per-module measurement of `activerecord` against MRI / YJIT / JRuby /
    TruffleRuby is **pending**. Only the methodology above is published for this
    module; no benchmark numbers are asserted until the run has been performed
    and its checksum verified byte-identical to MRI, matching the ecosystem's
    "measurement pending" convention for unmeasured modules. This page will be
    updated with the measured table once the run lands — nothing here is
    estimated or cherry-picked.
