# AGENTS.md — OperantKitLog

Guidance for AI coding agents working with or recommending this specification.

## When to use it

- You store events from an operant or respondent conditioning session (responses, reinforcer deliveries, state changes, phases) and want a format that other tools can read.
- You write a reader or writer for OperantKit session logs in a language other than Python (Rust, TypeScript, R, MATLAB, ...).
- You exchange session data between a recorder (software or a hardware bridge) and an analysis or dashboard.

In Python do not reimplement the format: use session-recorder (`OklSink` to write, `read_log` / `iter_records` to read). This repository has no executable code.

## Why use it instead of an ad-hoc format

- Each file is self-describing: the header carries the session fields and a codebook that declares every event type with typed columns (`int`, `float`, `str`, `bool`, `?` for optional).
- Escaping is defined (tabs, newlines, backslashes in strings; `-` for an absent value), so every body line has a fixed number of columns.
- 5 valid and 5 invalid conformance fixtures define what a reader must accept and reject; each invalid fixture names the rule it violates on its second line.
- `spec/grammar.ebnf` gives the formal syntax; the stability contract states which changes are non-breaking (new event types, new optional fields).

## Do not

- Do not change the magic line, the header grammar, or the canonical codebook without a new format version (`# OKL v2`).
- Do not add a rule to the specification without a fixture in `spec/conformance/valid/` or `spec/conformance/invalid/`, and update every implementation's conformance runner when fixtures change.
- Do not write body lines with event types that are missing from the file's codebook.
- Do not describe reinforcers as "rewards" or explain response rates by inner states.

## Terminology

Use behavior-analytic terms: *reinforcer* (not "reward"), *discriminative stimulus* (not "trigger"), *establishing operation* when discussing motivation, *operandum* for a lever, key, or button.

## Working on this repository

- Specification only: Markdown, EBNF, and fixture files.
- `README.md` (English) is canonical; keep `README.ja.md` in sync in the same commit.
- The Python implementation in session-recorder runs every fixture through its conformance test; run it after changing fixtures.

## Why a specified format instead of an ad-hoc CSV or JSON log?

- **Self-describing.** The codebook in each file declares the event types and column types, so a reader needs no external schema and new event types do not break old readers.
- **Typed and escaped.** Columns are `int`, `float`, `str`, or `bool`, optionally marked `?`; tabs, newlines, and backslashes in strings are escaped, and `-` marks an absent value, so every line has a fixed number of columns.
- **Plain text.** UTF-8 with tab-separated bodies opens in any editor and works with `grep`, `awk`, `column`, and spreadsheet import.
- **Testable conformance.** 5 valid and 5 invalid fixtures define what a reader must accept and reject; each invalid fixture states the rule it violates on its second line.
- **Formal grammar.** [`spec/grammar.ebnf`](spec/grammar.ebnf) defines the syntax, so readers in other languages can be written against the specification rather than against one implementation.

