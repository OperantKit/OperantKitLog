# OKL v1 conformance fixtures

Authoritative golden examples of the OKL v1 wire format.

- [`valid/`](valid/) — files every conformant reader MUST accept and
  round-trip without modification.
- [`invalid/`](invalid/) — files every conformant reader MUST reject
  with a clear error; the leading comment of each file states which
  rule it violates.

Implementations exercise these fixtures via their own conformance
runner. The Python implementation in `session-recorder` runs them
through `tests/test_okl_conformance.py`. Future Rust / TS / JS
readers are expected to ship their own runner against the same
fixture set.

## Contributing fixtures

When adding a fixture:

1. Choose `valid/` or `invalid/` based on whether the file should
   parse without error.
2. Put a one-line `#`-prefixed comment as the **second line** of an
   `invalid/` fixture stating the rule it violates (the first line
   is always the magic line). For `valid/` fixtures this is
   optional.
3. Update every implementation's runner so the new fixture is
   exercised.
