# OKL v1 — Wire format

> Status: Stable
> Canonical reference for the OKL v1 (OperantKitLog v1) wire format.
> Implementations MUST conform; the conformance fixtures under
> [`conformance/`](conformance/) are authoritative golden examples.

OKL v1 is a plain UTF-8 text file with the `.txt` extension. It
consists of three parts in this order:

1. A **magic line** identifying the format and version.
2. A **header block** carrying session metadata and the event
   *codebook*, terminated by a single `# ---` line.
3. A **body** of one TAB-separated event line per record.

```
# OKL v1                                ← magic
# session_name = "demo"                 ┐
# clock_type = "ManualClock"            │
# session_start = 0.0                   │ header
# events:                               │
#   response : id:int operandum:int?    │
# ---                                   ┘ terminator
0.0	response	1	-                      ┐
1.5	response	2	0                      │ body
```

See [`grammar.ebnf`](grammar.ebnf) for the formal grammar.

## 1. File-level requirements

- **Encoding**: UTF-8 without BOM. Files starting with a UTF-8 BOM
  (`U+FEFF`) are invalid; readers MUST raise a clear error.
- **Newlines**: writers MUST emit LF (`U+000A`). Readers MUST accept
  both LF and CRLF (strip trailing `\r` before parsing).
- **Extension**: `.txt`. Other extensions are not part of this spec
  but readers MAY accept them when the magic line is present.

## 2. Magic line

```
# OKL v1
```

The first line of every OKL v1 file. A reader that sees a different
magic line (e.g. `# OKL v2`) MUST refuse to parse the file.

## 3. Header block

A sequence of `#`-prefixed lines, terminated by exactly:

```
# ---
```

The header carries:

- **Top-level key/value lines** (`# key = value`) for required and
  optional session metadata.
- **Section blocks** introduced by `# meta:`, `# notes:`, `# events:`.
  Continuation lines under each section are written with two extra
  spaces of indent (which after lstrip leaves a normal `key = value`
  line, a quoted note string, or a codebook declaration).

Detailed rules: [`header.md`](header.md).

The `events:` section MUST declare every event type that appears in
the body. This is what makes the file *self-describing*.

Detailed rules: [`codebook.md`](codebook.md).

## 4. Body

After the terminator, every non-empty line is a body record:

```
<timestamp><TAB><type_name>[<TAB><col>...]
```

- `<timestamp>`: floating-point seconds since `session_start`. The
  Python writer emits `repr(float(t))`; readers MUST accept any
  finite-float spelling that `float(...)` parses.
- `<type_name>`: an identifier matching exactly one entry in the
  header `events:` codebook.
- `<col>...`: positional payload columns, ordered as declared in the
  codebook for that type. The column count MUST match.

Each column is encoded according to its declared field type:

| Field type | Encoding                                                                      |
|------------|-------------------------------------------------------------------------------|
| `int`      | base-10 signed integer (`-?[0-9]+`)                                           |
| `float`    | finite-float spelling (`1.5`, `1e-3`, `-1.0`, ...)                            |
| `bool`     | `true` or `false` (lowercase)                                                 |
| `str`      | TSV-escaped string (see §5)                                                   |
| optional   | a missing value is encoded as a single `-` (the *absent* token)               |

A field declared optional (suffixed `?` in the codebook) MAY be `-`;
a field NOT declared optional MUST NOT be `-`.

## 5. TSV escaping (string body columns)

A string body column escapes TAB, LF, CR, backslash, and the literal
single-character `-`:

| Raw     | Encoded |
|---------|---------|
| `\`     | `\\`    |
| TAB     | `\t`    |
| LF      | `\n`    |
| CR      | `\r`    |
| `-`     | `\-`    |

A literal `--` is encoded as `--` (only the *single-character* string
`-` collides with the absent token). Unknown `\X` escapes pass
through verbatim on decode.

This encoding is distinct from the TOML basic-string escapes used in
the header. Do not mix.

## 6. Stability contract

- The magic line, header grammar, canonical codebook, and body
  encoding rules are stable. Breaking changes require a new format
  version (`# OKL v2`) and a transition window during which both
  versions parse.
- Adding a **new event type** to the canonical codebook is a
  non-breaking change.
- Adding a **new optional field** to an existing event type is a
  non-breaking change because every consumer reads the codebook from
  the file's own header, not from a hard-coded schema.

## 7. Conformance

[`conformance/`](conformance/) contains the authoritative golden
fixtures. Implementations MUST:

- accept every file under `conformance/valid/` and round-trip it;
- reject every file under `conformance/invalid/` with a clear error.

The Python implementation in `session-recorder/tests/test_okl_conformance.py`
exercises this directly; future implementations (Rust, TS, ...) MUST
ship their own conformance runner.
