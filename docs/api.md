# Usage & API

The public API lives at the module root (`github.com/go-ruby-xslt/xslt`). It is
**Ruby-shaped but Go-idiomatic**: `Transform` / `Apply` mirror
`Nokogiri::XSLT(ss).transform` / `.apply_to`, while the surface follows Go
conventions — an explicit `error`, value types, no global state.

!!! success "Status: implemented"
    The library is built and importable as `github.com/go-ruby-xslt/xslt`, bound
    into `rbgo` as a native module; see [Roadmap](roadmap.md).

## Install

```sh
go get github.com/go-ruby-xslt/xslt
```

## Worked example

```go
import (
    "github.com/go-ruby-nokogiri/nokogiri"
    "github.com/go-ruby-xslt/xslt"
)

ss, _ := xslt.ParseString(stylesheetXML)   // compile the stylesheet once
doc, _ := nokogiri.XML(sourceXML)           // parse the source document

result, _ := ss.Transform(doc, nil)         // -> *nokogiri.Document (result tree)
out, _    := ss.Apply(doc, nil)             // -> serialized string (xsl:output honoured)
```

A compiled stylesheet can be applied to any number of source documents. `Apply`
serializes the result tree honouring the stylesheet's `xsl:output` (`method` =
xml / html / text, `indent`, `encoding`, `omit-xml-declaration`, `standalone`,
doctype, `cdata-section-elements`).

## Shape

```go
// ParseString compiles an XSLT 1.0 stylesheet from its XML source.
func ParseString(stylesheetXML string) (*Stylesheet, error)

// Transform applies the compiled stylesheet to a source document and
// returns the result tree.
func (ss *Stylesheet) Transform(doc *nokogiri.Document, params map[string]any) (*nokogiri.Document, error)

// Apply applies the stylesheet and returns the serialized string,
// honouring xsl:output.
func (ss *Stylesheet) Apply(doc *nokogiri.Document, params map[string]any) (string, error)
```

## Stylesheet parameters

Top-level `xsl:param` values are overridable by the caller via the `params`
argument — a `map[string]any` whose values are:

| Go type | XSLT value |
| --- | --- |
| `string` | string |
| `float64` | number |
| `bool` | boolean |
| `*nokogiri.NodeSet` | node-set |

```go
out, _ := ss.Apply(doc, map[string]any{
    "title":     "Report",   // string
    "threshold": 3.0,        // number
    "verbose":   true,       // boolean
})
```

## Relationship to Ruby

This mirrors the Ruby surface:

```ruby
Nokogiri::XSLT(stylesheet).transform(doc)   # -> result document   == ss.Transform
Nokogiri::XSLT(stylesheet).apply_to(doc)    # -> serialized string == ss.Apply
```

`go-ruby-xslt/xslt` is **standalone and reusable**, and is the backend bound into
[go-embedded-ruby](https://github.com/go-embedded-ruby/ruby) by `rbgo` as a
native module — the same way [go-ruby-regexp](https://github.com/go-ruby-regexp)
and [go-ruby-erb](https://github.com/go-ruby-erb) are bound. The dependency runs
the other way: this library has no dependency on the Ruby runtime.

## Conformance

Correctness is defined by the XSLT 1.0 spec and reference Ruby. Deterministic
golden vectors (stylesheet + source → expected output, drawn from the
[XSLT 1.0](https://www.w3.org/TR/1999/REC-xslt-19991116) spec) hold coverage
with **no Ruby present**; a **differential oracle** against `Nokogiri::XSLT`
(libxslt) runs where a new-enough Ruby is available. The oracle tests skip
themselves where `ruby` is not on `PATH` (e.g. the qemu arch lanes), so the
cross-arch builds still validate the library.

## Deferred

This is a **1.0** processor. XSLT 2.0 / XPath 2.0 (sequences, `xsl:function`,
`xsl:for-each-group`, schema awareness, tunnel params) is out of scope.
`document()` of an external URI and multi-document `xsl:import` / `xsl:include`
need a URI resolver and are deferred (they compile to no-ops so stylesheets that
reference them still load); `disable-output-escaping` is accepted but emits
normally.
