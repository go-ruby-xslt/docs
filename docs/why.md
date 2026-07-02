# Why pure Go

`go-ruby-xslt/xslt` implements Ruby's `Nokogiri::XSLT` in **pure Go, with cgo
disabled**. XSLT 1.0 transformation is **deterministic**: given a stylesheet, a
source document and its parameters, the result tree is a pure function of those
inputs — no live binding, no evaluation of arbitrary Ruby. That is exactly the
part that can — and should — live as a standalone Go library.

## Replacing libxslt with a pure-Go path

`Nokogiri::XSLT` is normally a thin Ruby wrapper over the C library **libxslt**
(itself over libxml2). That C dependency is what forces cgo, a C toolchain, and
per-platform builds. This module removes it: it is a **from-scratch pure-Go XSLT
1.0 engine** that compiles and applies stylesheets over the pure-Go XML DOM and
XPath 1.0 engine of
[go-ruby-nokogiri](https://github.com/go-ruby-nokogiri/nokogiri). So:

- any Go program can import `github.com/go-ruby-xslt/xslt` directly and transform
  XML with no libxslt and no Ruby runtime;
- the dependency runs the *other* way — `rbgo` binds this module as a native
  module (the same pattern as [go-ruby-regexp](https://github.com/go-ruby-regexp)
  and [go-ruby-erb](https://github.com/go-ruby-erb)), rather than this module
  depending on the interpreter;
- the behaviour is pinned by golden vectors drawn from the XSLT 1.0 spec plus a
  **differential oracle** against `Nokogiri::XSLT` (libxslt), independent of any
  one consumer.

This is **not** an extraction of an rbgo prelude — it is a purpose-built XSLT
engine layered on go-ruby-nokogiri's DOM and XPath.

## Built on go-ruby-nokogiri

The XPath 1.0 engine is deliberately **not** reimplemented here. The processor
drives go-ruby-nokogiri's engine through its extension seam (`XPathContext`):
XSLT variable bindings (`$name`), an extension-function resolver for the XSLT
function library, and XSLT `current()` semantics. XSLT and XPath stay one engine,
not two divergent copies.

## Why pure Go matters here

Because the library is CGO-free and depends only on go-ruby-nokogiri, it:

- cross-compiles to every Go target with no C toolchain and links into a single
  static binary — no libxslt/libxml2 to build or ship;
- has **no dependency on the Ruby runtime** — the dependency runs the other way;
- can be differentially tested against `Nokogiri::XSLT` wherever a new-enough
  `ruby` is on `PATH`, while the cross-arch lanes (where `ruby` is absent) still
  validate the library against deterministic golden vectors.

See [Usage & API](api.md) for the surface and [Roadmap](roadmap.md) for what is
in scope.
