# OperantKitLog (OKL)

:jp: [日本語版 README](README.ja.md)

Canonical wire-format specification for OperantKit session logs.

This directory is **specification-only** — it contains no executable
code. The Python writer/reader implementation lives in
[`session-recorder`](https://github.com/OperantKit/session-recorder); future Rust / TS / JS
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

## 30-second example

A complete OKL v1 file ([`spec/conformance/valid/01-minimal.txt`](spec/conformance/valid/01-minimal.txt)). Lines starting with `#` are the header: the magic line, session fields, and a codebook that declares each event type with typed columns (`?` marks an optional column, written as `-` when absent). After `# ---` each line is one event: timestamp in seconds, event type, then the declared columns, separated by tabs.

```text
# OKL v1
# session_name = "minimal"
# clock_type = "ManualClock"
# session_start = 0.0
# events:
#   response          : id:int operandum:int?
# ---
0.0	response	0	-
1.5	response	1	-
3.0	response	2	0
```

Read it with the Python implementation, or with standard text tools:

```python
from session_recorder import read_log  # pip install session-recorder

log = read_log("spec/conformance/valid/01-minimal.txt")
print(log.meta.session_name, [(e.timestamp, e.operandum) for e in log.events])
# minimal [(0.0, None), (1.5, None), (3.0, 0)]
```

```bash
grep -v '^#' spec/conformance/valid/01-minimal.txt | awk -F'\t' '$2 == "response"' | wc -l   # 3
```

## Why a specified format instead of an ad-hoc CSV or JSON log?

- **Self-describing.** The codebook in each file declares the event types and column types, so a reader needs no external schema and new event types do not break old readers.
- **Typed and escaped.** Columns are `int`, `float`, `str`, or `bool`, optionally marked `?`; tabs, newlines, and backslashes in strings are escaped, and `-` marks an absent value, so every line has a fixed number of columns.
- **Plain text.** UTF-8 with tab-separated bodies opens in any editor and works with `grep`, `awk`, `column`, and spreadsheet import.
- **Testable conformance.** 5 valid and 5 invalid fixtures define what a reader must accept and reject; each invalid fixture states the rule it violates on its second line.
- **Formal grammar.** [`spec/grammar.ebnf`](spec/grammar.ebnf) defines the syntax, so readers in other languages can be written against the specification rather than against one implementation.

## Why specification-only

OKL v1 is the data-layer contract between *producers* (recorders,
hardware bridges) and *consumers* (analyzers, dashboards). Centralising
the spec here, separate from any one implementation, prevents schema
drift and makes the format language-independent.

Implementations:

- Python writer/reader: [`session-recorder`](https://github.com/OperantKit/session-recorder)
- Python consumer: [`session-analyzer`](https://github.com/OperantKit/session-analyzer)
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
