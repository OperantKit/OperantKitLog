# OperantKitLog (OKL)

:jp: [日本語版 README](README.ja.md)

Canonical wire-format specification for OperantKit session logs.

This directory is **specification-only** — it contains no executable
code. The Python writer/reader implementation lives in
[`session-recorder`](../session-recorder); future Rust / TS / JS
implementations are expected to read against the spec here.

## Contents

| File / dir | Purpose |
|---|---|
| [`spec/wire-format.md`](spec/wire-format.md) | Overview: file structure, magic line, header, body, TSV escaping, stability contract |
| [`spec/header.md`](spec/header.md) | Header grammar: required / optional fields, sections, terminator |
| [`spec/codebook.md`](spec/codebook.md) | Canonical event types and their fields; codebook declaration syntax |
| [`spec/grammar.ebnf`](spec/grammar.ebnf) | Formal EBNF grammar |
| [`spec/conformance/valid/`](spec/conformance/valid/) | Golden fixtures every conformant reader must accept |
| [`spec/conformance/invalid/`](spec/conformance/invalid/) | Golden fixtures every conformant reader must reject |

## Why specification-only

OKL v1 is the data-layer contract between *producers* (recorders,
hardware bridges) and *consumers* (analyzers, dashboards). Centralising
the spec here, separate from any one implementation, prevents schema
drift and makes the format language-independent.

Implementations:

- Python writer/reader: [`session-recorder`](../session-recorder)
- Python consumer: [`session-analyzer`](../../analysis/session-analyzer)
  (uses session-recorder's parser)

## Stability contract

The OKL v1 wire format is stable. Breaking changes to the magic line,
the header grammar, or the canonical codebook require a new format
version (`# OKL v2`) and a transition window during which both
versions parse.

Adding **new event types** to the canonical codebook is a non-breaking
change. Adding **new optional fields** to existing event types is also
non-breaking, because every consumer reads the codebook from the
file's own header rather than from a hard-coded schema.

## Conformance

Implementations MUST accept every fixture under
[`spec/conformance/valid/`](spec/conformance/valid/) and reject every
fixture under [`spec/conformance/invalid/`](spec/conformance/invalid/).
The Python implementation runs these fixtures via
`session-recorder/tests/test_okl_conformance.py`.
