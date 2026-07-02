# Roadmap

`go-ruby-xslt/xslt` is grown **test-first**, each capability validated against
golden vectors from the XSLT 1.0 spec and, where Ruby is present, differentially
against `Nokogiri::XSLT` (libxslt) — rather than built in isolation. The XSLT 1.0
processor is **complete**.

| Stage | What | Status |
| --- | --- | --- |
| Stylesheet parsing & compile | `xsl:stylesheet` / `xsl:transform` and literal-result-element stylesheets parsed and compiled once with `ParseString`, ready to apply to any number of source documents. | **Done** |
| Template rules & conflict resolution | Template rules by `match` / `name` / `priority` / `mode`, resolved by import precedence → priority → document order, with the built-in default templates for every node kind and `xsl:apply-imports`. | **Done** |
| Instructions | `xsl:apply-templates` (`select`, `mode`), `xsl:call-template` / `xsl:with-param`, `xsl:value-of`, `xsl:for-each`, `xsl:if`, `xsl:choose` / `when` / `otherwise`. | **Done** |
| Sorting, copy & number | `xsl:sort` (data-type, order, multiple keys), `xsl:copy` / `xsl:copy-of` (deep node-set and RTF copy), and `xsl:number` (single / any level; `1`, `01`, `a`, `A`, `i`, `I`). | **Done** |
| Variables, params & keys | `xsl:variable` and `xsl:param` (top-level and local, caller-overridable), `xsl:key` + `key()`, `xsl:decimal-format` + `format-number()`, resolved through go-ruby-nokogiri's XPath extension seam. | **Done** |
| Result construction & output | `xsl:element` / `attribute` / `text` / `comment` / `processing-instruction`, attribute-value templates and `use-attribute-sets`; `xsl:output` (method xml / html / text, indent, encoding, doctype, `omit-xml-declaration`, `cdata-section-elements`). | **Done** |
| Function library | `key`, `format-number`, `current`, `generate-id`, `system-property`, `element-available`, `function-available`, `unparsed-entity-uri`. | **Done** |
| Golden vectors & differential oracle | Deterministic golden vectors from the XSLT 1.0 spec hold coverage with no Ruby present; a differential oracle against `Nokogiri::XSLT` (libxslt) runs where a new-enough Ruby is available. 100% coverage, gofmt + go vet clean, green across all six 64-bit Go arches and three OSes. | **Done** |

## Documented out-of-scope boundaries

These are **deliberate**, recorded so the module's surface is unambiguous:

- **XSLT 1.0 only.** XSLT 2.0 / XPath 2.0 (sequences, `xsl:function`,
  `xsl:for-each-group`, schema awareness, tunnel params) is out of scope.
- **No external URI resolver.** `document()` of an external URI and
  multi-document `xsl:import` / `xsl:include` are deferred — they compile to
  no-ops so stylesheets that reference them still load. `disable-output-escaping`
  is accepted but emits normally.
- **Built on go-ruby-nokogiri, not a second XPath engine.** The XPath 1.0 engine
  is go-ruby-nokogiri's, driven through its extension seam — not reimplemented
  here.
- **A reusable library, not the interpreter.** The module implements the
  deterministic transform; anything that needs a live Ruby binding is the
  consumer's job — that is why `rbgo` binds this module rather than the reverse.

See [Usage & API](api.md) for the surface and [Why pure Go](why.md) for the
libxslt-replacement rationale.
