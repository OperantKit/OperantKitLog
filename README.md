# OperantKitLog (OKL)

:jp: [日本語版 README](README.ja.md)

OperantKitLog (OKL) is the file format in which [OperantKit](https://github.com/OperantKit) records an operant or respondent conditioning session: every response, reinforcer delivery, state change, and phase, one line per event. This repository is the canonical wire-format specification for OperantKit session logs. It is for anyone who reads, writes, or converts these logs, including people who write a reader in R, MATLAB, or another language.

- **Plain text.** A short header, then one tab-separated line per event; it opens in any text editor and works with `grep`, `awk`, and spreadsheet import.
- **Self-describing.** The header declares each event type and the type of each column, so no outside schema is needed.
- **Checkable.** Example files define what a reader must accept and what it must reject.

This repository is **specification-only**: it contains no executable code. Notes for AI coding agents are in [AGENTS.md](AGENTS.md).

## Installation

Nothing to install. To read and write OKL files from Python, use [session-recorder](https://github.com/OperantKit/session-recorder); its README gives the install command.

## Example

A complete OKL v1 file, [`spec/conformance/valid/01-minimal.txt`](spec/conformance/valid/01-minimal.txt):

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

Lines starting with `#` are the header: the first line names the format and version, then come the session fields and a codebook that declares each event type with typed columns. `?` marks an optional column, written as `-` when it is empty; here `operandum` (the lever or key the response was made on) is given only for the third response. After `# ---`, each line is one event: time in seconds, event type, then the declared columns, separated by tabs.

Read it with session-recorder:

```python
from session_recorder import read_log

log = read_log("spec/conformance/valid/01-minimal.txt")
print(log.meta.session_name, [(e.timestamp, e.operandum) for e in log.events])
# minimal [(0.0, None), (1.5, None), (3.0, 0)]
```

or with standard text tools:

```bash
grep -v '^#' spec/conformance/valid/01-minimal.txt | awk -F'\t' '$2 == "response"' | wc -l   # 3
```

## What is inside

| File / dir | Purpose |
|---|---|
| [`spec/wire-format.md`](spec/wire-format.md) | Overview: file structure, magic line, header, body, TSV escaping, stability contract |
| [`spec/header.md`](spec/header.md) | Header grammar: required / optional fields, sections, terminator |
| [`spec/codebook.md`](spec/codebook.md) | The 7 canonical event types and their fields; codebook declaration syntax |
| [`spec/grammar.ebnf`](spec/grammar.ebnf) | Formal EBNF grammar |
| [`spec/conformance/valid/`](spec/conformance/valid/) | 5 example files every conformant reader must accept |
| [`spec/conformance/invalid/`](spec/conformance/invalid/) | 5 example files every conformant reader must reject; the second line of each states the rule it breaks |

OKL v1 is the contract between programs that produce session logs (recorders, hardware bridges) and programs that consume them (analyses, dashboards). Keeping the specification apart from any one implementation keeps the format language-independent.

Implementations:

- Python writer / reader: [session-recorder](https://github.com/OperantKit/session-recorder). Its tests run every file under `spec/conformance/`.
- Python consumer: [session-analyzer](https://github.com/OperantKit/session-analyzer) (uses session-recorder's parser).

### Stability contract

The OKL v1 wire format is stable. Breaking changes to the magic line, the header grammar, or the canonical codebook require a new format version (`# OKL v2`) and a transition window during which both versions parse.

Adding **new event types** to the canonical codebook is a non-breaking change. Adding **new optional fields** to existing event types is also non-breaking, because every consumer reads the codebook from the file's own header rather than from a hard-coded schema.

### Conformance

Implementations MUST accept every file under [`spec/conformance/valid/`](spec/conformance/valid/) and reject every file under [`spec/conformance/invalid/`](spec/conformance/invalid/). See [`spec/conformance/README.md`](spec/conformance/README.md) for how to add a fixture.

## Limitations

- The only implementation is in Python ([session-recorder](https://github.com/OperantKit/session-recorder)). A reader in another language has to be written against this specification.
- The specification names `.txt` as the file extension; readers may accept other extensions when the magic line `# OKL v1` is present.

## License

This repository contains no license file.
