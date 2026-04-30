# OKL v1 — Event codebook

> Status: Stable
> Companion to [`wire-format.md`](wire-format.md) and
> [`header.md`](header.md). Specifies the canonical event types and
> their fields, and how a file declares its own codebook.

## 1. Codebook declaration

The header `events:` section declares, for every event type the file
contains, the names and types of its positional payload columns.

```
# events:
#   response          : id:int operandum:int?
#   reinforcer_start  : id:int potency:float operandum:int?
#   reinforcer_end    : id:int operandum:int?
```

Each codebook line has the form:

```
<type_name> : <field_decl> [ <field_decl> ... ]
```

with each `<field_decl>` being:

```
<identifier> : <field_type> [?]
```

The `?` suffix marks the field optional. An optional field MAY appear
as the absent token `-` in body rows; a non-optional field MUST NOT.

Field types: `int`, `float`, `str`, `bool`. See
[`wire-format.md` §4](wire-format.md#4-body) for column encodings.

Codebook field order in the header determines the column order in the
body. Writers and readers MUST agree on this ordering through the
codebook itself, never through an out-of-band schema.

## 2. Canonical types

The following event types are *canonical* in OKL v1: every conformant
writer that emits events of these kinds MUST use the names and field
shapes below.

### `response`

A discrete operant response.

| Field       | Type   | Optional? | Meaning                                            |
|-------------|--------|-----------|----------------------------------------------------|
| `id`        | int    | no        | Sequential response id (0-based, monotonic)        |
| `operandum` | int    | yes       | Operandum index for multi-operandum sessions       |

### `reinforcer_start`

The onset of a reinforcer delivery.

| Field       | Type   | Optional? | Meaning                                            |
|-------------|--------|-----------|----------------------------------------------------|
| `id`        | int    | no        | Sequential reinforcer id (0-based, monotonic)      |
| `potency`   | float  | no        | Reinforcer potency (1.0 by default)                |
| `operandum` | int    | yes       | Operandum index for multi-operandum sessions       |

### `reinforcer_end`

The offset of a reinforcer delivery (matching a previous `reinforcer_start`
by `id`).

| Field       | Type   | Optional? | Meaning                                            |
|-------------|--------|-----------|----------------------------------------------------|
| `id`        | int    | no        | Reinforcer id (matches the corresponding `reinforcer_start`) |
| `operandum` | int    | yes       | Operandum index for multi-operandum sessions       |

### `state_change`

A session-level state transition (e.g., `IDLE → RUNNING`).

| Field  | Type | Optional? | Meaning                |
|--------|------|-----------|------------------------|
| `from` | str  | no        | State name (`IDLE`, `RUNNING`, ...) |
| `to`   | str  | no        | State name              |

### `component_change`

A change of compound-schedule component (e.g., the active member of a
multiple / mix / chain / tand schedule).

| Field  | Type | Optional? | Meaning                |
|--------|------|-----------|------------------------|
| `from` | str  | no        | Component name          |
| `to`   | str  | no        | Component name          |

### `phase_enter`

The start of a phase, optionally annotated with a context (per the
`contingency-dsl` foundations of context).

| Field      | Type | Optional? | Meaning                                     |
|------------|------|-----------|---------------------------------------------|
| `label`    | str  | no        | Human-readable phase label                  |
| `name`     | str  | yes       | Context name (`A`, `B`, `room_a`, ...)      |
| `time`     | str  | yes       | Temporal context coordinate                 |
| `location` | str  | yes       | Spatial context coordinate                  |
| `cue`      | str  | yes       | Stimulus context coordinate                 |

### `phase_exit`

The end of a phase. Pairs with the most recent `phase_enter` of the
same `label`.

| Field   | Type | Optional? | Meaning                  |
|---------|------|-----------|--------------------------|
| `label` | str  | no        | Phase label               |

## 3. Custom event types

A file MAY declare event types beyond the canonical set. Custom
declarations follow the same syntax (§1). A reader sees the
declarations in the header and parses unknown body rows accordingly.

A consumer that does not understand a custom type MAY:

- skip those rows silently, or
- skip them with a warning, or
- raise an error, depending on its own configured policy.

## 4. Stability

- Adding a **new event type** to §2 is non-breaking.
- Adding a **new optional field** to an existing canonical type is
  non-breaking (consumers parse columns by codebook position; new
  optional columns at the end appear as extra `field?` declarations
  in the codebook).
- Renaming, reordering, removing, or changing the type of an existing
  canonical field is BREAKING and requires a new format version
  (`# OKL v2`).
