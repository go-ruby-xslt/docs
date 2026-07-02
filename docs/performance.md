# Performance

`go-ruby-xslt/xslt` is the pure-Go library that
[`rbgo`](https://github.com/go-embedded-ruby/ruby) binds for Ruby's
`Nokogiri::XSLT`. This page records the **methodology** for a comparative
benchmark of that module against the reference Ruby runtimes, part of the
ecosystem-wide per-module parity suite.

## What is measured

The **same** Ruby script — a `Nokogiri::XSLT(ss).transform(doc)` /
`.apply_to(doc)` round-trip over a representative stylesheet and source document
— is run under every runtime. `rbgo`'s number reflects **this pure-Go library
doing the work** (over go-ruby-nokogiri's DOM/XPath); every other column is that
interpreter's own `nokogiri` / libxslt. So the comparison is the **Ruby-visible
operation**, apples-to-apples across interpreters. The script prints a
deterministic checksum and its output is checked **byte-identical to the
`Nokogiri::XSLT` (libxslt) oracle** before timing.

- **Method:** best-of-5 wall time (best, not mean, to suppress scheduler noise);
  single-shot processes, no warm-up beyond the script's own loop.
- **Runtimes (planned):** `ruby +PRISM` (MRI, the oracle) and `ruby --yjit`;
  `jruby` (OpenJDK); `truffleruby` (GraalVM CE Native).
- The benchmark script and harness live in rbgo's repo under
  [`bench/modules/`](https://github.com/go-embedded-ruby/ruby/tree/main/bench/modules)
  (`xslt.rb` + `run.sh`). Reproduce:
  `RBGO=./rbgo TRUFFLE=truffleruby bash bench/modules/run.sh 5`.

## Results

!!! note "Measurement pending"
    Numbers for this module have **not yet been measured** on a controlled host.
    Rather than publish fabricated figures, the results table is intentionally
    omitted until a real best-of-5 run against MRI (and the other runtimes) is
    recorded here. The methodology above is fixed; only the measured numbers are
    outstanding.

    When the run lands, every row will be a real measured number — the same
    single-shot `ruby file.rb` cost as `rbgo` and MRI, not steady-state JIT
    figures — with the byte-identical-to-oracle check enforced before timing.
