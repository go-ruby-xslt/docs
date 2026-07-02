# go-ruby-xslt documentation

**A pure-Go XSLT 1.0 processor — Nokogiri::XSLT without libxslt, no cgo.**

`go-ruby-xslt/xslt` is a pure-Go (zero cgo) **XSLT 1.0 processor** — the deferred
XSLT layer of Ruby's [Nokogiri](https://nokogiri.org). `Nokogiri::XSLT` is
normally a C wrapper over **libxslt**; this module instead compiles and applies
XSLT 1.0 stylesheets over the pure-Go XML DOM and XPath 1.0 engine of
[go-ruby-nokogiri](https://github.com/go-ruby-nokogiri/nokogiri), so the whole
transformation path is CGO-free and cross-compiles to every supported arch. The
module path is `github.com/go-ruby-xslt/xslt`.

It is the XSLT backend for [go-embedded-ruby](https://github.com/go-embedded-ruby/ruby)
(`rbgo`), built **from scratch** on go-ruby-nokogiri, but is a **standalone,
reusable** Go module — a sibling of
[go-ruby-regexp](https://github.com/go-ruby-regexp),
[go-ruby-erb](https://github.com/go-ruby-erb) and
[go-ruby-nokogiri](https://github.com/go-ruby-nokogiri) (the DOM/XPath core it
builds on). The dependency runs the other way: this library has **no dependency
on the Ruby runtime**.

!!! success "Status: a full XSLT 1.0 processor"
    Stylesheet parsing and compile; template rules (`match` / `name` / `priority` / `mode`) with conflict resolution by import precedence → priority → document order, the built-in default templates and `xsl:apply-imports`; the instruction set (`apply-templates`, `call-template`, `value-of`, `for-each`, `if`, `choose`); `xsl:sort`, `copy` / `copy-of`, `xsl:number`; variables, params, keys and `format-number()`; result-element construction and `xsl:output` (xml / html / text). Validated by golden vectors from the XSLT 1.0 spec plus a **differential oracle** against `Nokogiri::XSLT` (libxslt), at 100% coverage, `gofmt` + `go vet` clean, CI green across the six 64-bit Go targets and three OSes.

## Quick taste

```go
import (
    "github.com/go-ruby-nokogiri/nokogiri"
    "github.com/go-ruby-xslt/xslt"
)

ss, _ := xslt.ParseString(stylesheetXML)   // compile once
doc, _ := nokogiri.XML(sourceXML)           // parse the source

result, _ := ss.Transform(doc, nil)         // -> *nokogiri.Document (result tree)
out, _    := ss.Apply(doc, nil)             // -> serialized string (xsl:output honoured)
```

## Repositories

| Repo | What it is |
| --- | --- |
| [`xslt`](https://github.com/go-ruby-xslt/xslt) | the library — a pure-Go XSLT 1.0 processor over go-ruby-nokogiri |
| [`docs`](https://github.com/go-ruby-xslt/docs) | this documentation site (MkDocs Material, versioned with mike) |
| [`go-ruby-xslt.github.io`](https://github.com/go-ruby-xslt/go-ruby-xslt.github.io) | the organization landing page (Hugo) |
| [`brand`](https://github.com/go-ruby-xslt/brand) | logo and brand assets |

## Principles

- **Pure Go, `CGO_ENABLED=0`** — trivial cross-compilation, a single static
  binary, no C toolchain and no libxslt to link.
- **Built on go-ruby-nokogiri.** The XPath 1.0 engine is not reimplemented; the
  processor drives go-ruby-nokogiri's DOM and XPath through its extension seam.
- **Standalone & reusable.** The XSLT backend for `rbgo`, built from scratch; no
  dependency on the Ruby runtime — the dependency runs the other way.
- **100% test coverage** is the target, enforced as a CI gate, across 6 arches
  and 3 OSes.

## Where to go next

- [Why pure Go](why.md) — why replacing libxslt with a pure-Go path over
  go-ruby-nokogiri is the right split.
- [Usage & API](api.md) — the public surface and worked examples.
- [Roadmap](roadmap.md) — what is done and what is deferred by design.

Source lives at [github.com/go-ruby-xslt/xslt](https://github.com/go-ruby-xslt/xslt).
