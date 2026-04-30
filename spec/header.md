# OKL v1 — Header

> Status: Stable
> Companion to [`wire-format.md`](wire-format.md). Specifies the
> grammar and required / optional fields of the OKL v1 header block.

The header runs from the magic line (exclusive) through the line
`# ---` (the terminator, inclusive).

## 1. Line forms

Every header line begins with `#`, optionally followed by a single
space. The body of the line (after the prefix) is one of:

| Form                          | Meaning                                  |
|-------------------------------|------------------------------------------|
| empty                         | blank header line, no semantic effect    |
| `key = <toml_value>`          | top-level metadata                       |
| `meta:` / `notes:` / `events:`| section markers                          |
| (indented continuation)       | belongs to the most recent section       |

Section continuation lines are written with two extra spaces of
indent. After stripping leading whitespace, the body of a continuation
line is interpreted by section:

- under `meta:` — a `key = <toml_value>` pair
- under `notes:` — a single TOML basic-string
- under `events:` — one codebook declaration ([`codebook.md`](codebook.md))

## 2. Required top-level fields

Every OKL v1 file MUST declare these three top-level keys; a missing
key is a parse error.

| Key             | TOML type | Meaning                                     |
|-----------------|-----------|---------------------------------------------|
| `session_name`  | string    | Human-readable session identifier           |
| `clock_type`    | string    | Name of the clock used for timestamps       |
| `session_start` | float     | Wall-clock time origin (seconds since epoch, or 0.0 for relative-time clocks) |

## 3. Optional top-level fields

| Key                 | TOML type | Meaning                                                |
|---------------------|-----------|--------------------------------------------------------|
| `subject_id`        | string    | Subject / participant identifier                       |
| `replication_index` | int       | 0-based replication / run index                        |
| `experiment_name`   | string    | Higher-level experiment label                          |
| `protocol_id`       | string    | Protocol / task version identifier                     |
| `task_file_hash`    | string    | Hash of the source DSL / task file                     |
| `wall_clock_start`  | string    | ISO-8601 timestamp (timezone-aware)                    |

These fields mirror `experiment_core.SessionMeta`. They are written
when not `None` and skipped otherwise.

## 4. Unknown top-level keys

Top-level keys not in the lists above SHOULD NOT be emitted by writers.
Readers MAY tolerate them by:

- preserving the key under the `metadata` dict so the file round-trips,
  and
- emitting a warning once per unknown key.

This policy keeps typos visible without making file parsing fail
hard. Strict readers MAY treat unknown keys as a parse error.

## 5. `meta:` section

A flat key/value table for free-form session metadata. Continuation
lines under `# meta:` are normal `key = <toml_value>` pairs:

```
# meta:
#   schedule_dsl = "FR3"
#   note_count = 0
#   replicate = true
```

Allowed value types: TOML basic-string, integer, float, boolean.

## 6. `notes:` section

A list of free-form post-hoc annotation strings. Continuation lines
under `# notes:` are TOML basic-strings:

```
# notes:
#   "subject lever-pressed during ITI"
#   "rig 2 was rebooted at t=420"
```

The order of notes MUST be preserved across round-trips.

## 7. `events:` section

The event codebook. See [`codebook.md`](codebook.md).

## 8. Terminator

The header ends with exactly the line:

```
# ---
```

A reader MUST accept any number of header lines preceding the
terminator (including zero non-required lines, which is technically
ill-formed because of §2 but must produce a clear error rather than
silent acceptance).
