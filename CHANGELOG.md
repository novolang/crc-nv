# Changelog

Newest first.  Below `1.0.0` a breaking change bumps the **minor**
number and a compatible one the **patch**; see [Version numbers in the
Orbit package registry](https://novo-lang.org/docs/registry/semver.html).

## 0.1.4 — 2026-09-08

- **Declares its layer**: `layer = "core"` in the manifest — the public API requires no effects, and `novo pkg publish` now checks the code against that budget.  No code changed.  The layers are described under Design in the [publishing guide](https://novo-lang.org/docs/publishing.html#design).

## 0.1.3

Documentation: the reference is generated from the code, and the
examples in it are doctests.  No code changed — every check value is
what 0.1.2 computed.

- **Every `pub` item is documented under Go's rule**, the comment block
  directly above the declaration, its first sentence the summary a
  reader meets before opening anything.  The methods of `impl Crc` carry
  their own.  `novo doc` turns that into
  [the package's page](https://novo-lang.org/packages/crc-nv).
- **Eight worked examples, and they run.**  Each constructor computes
  its own published check value over `123456789`, and the streaming
  form is shown reaching the same answer as the one-call form.  A
  fenced `novo` block in a documentation comment is compiled by
  `novo doc` and run by `novo test src/crc.nv`, so a check value that
  stopped being true is a failing test.
- **The README's algorithm table is gone.**  The parameters and the
  check values are on the generated page, where they are read out of
  the code that computes them; the README keeps what a generator cannot
  say — which format uses which, and why the misleading name is kept.

## 0.1.2

Developed in its own repository from this version.  `novolang/crc-nv` is
where the sources live, where CI runs and where releases are tagged;
the novo-lang monorepo no longer carries a copy.  No code changed —
every signature and every byte on the wire is what 0.1.1 shipped.

- **`LICENSE` ships with the package.**  It is on the publish
  allow-list, so the tarball now carries the Apache-2.0 text rather
  than only naming it in the manifest.

## 0.1.1

A patch: every check value is the value it was and no signature moved.
The four published check values over `"123456789"`, the agreement with
`std.codec`'s C implementations, and the streaming and range forms all
still pass, which is what says so.

- **The table builders and the step are written with the operators.**
  Twenty-five `bits.*` calls become `&`, `^`, `<<` and `>>>`, and the
  reflected table's shift-only branch is `remainder >>>= 1`.  The
  reflected step is now two lines a reader can check against the
  algorithm — `(state ^ byte) & 0xff` selects the table entry and
  `self.table[index] ^ (state >>> 8)` shifts the rest of the state into
  it.  Every shift count that is an expression is given a name, because
  the shifts bind looser than `-` and the inline form reads wrong.
  `std.bits` is no longer imported.
- **The test module moved out of `src/`.**  A package's `src/` ships
  whole and a consumer compiles every module in it, so the suite is
  under `tests/` where it is not published.  Run it with
  `novo test tests/crc_tests.nv`.

## 0.1.0

First release: `crc16_ccitt_false`, `crc16_modbus`, `crc32`, `crc32c`,
and the `Crc` value's `start` / `step` / `update` / `update_range` /
`finish` / `checksum` / `verify`.

- **Four fully specified algorithms.**  Polynomial, initial state,
  reflection and final xor are all named, and each is asserted against
  the check value its catalogue entry publishes.
- **Streaming is the primitive.**  The running state is an `Int`, so a
  message that arrives in pieces gives the same answer as one that
  arrived whole; `checksum` is the one-call form on top.
- **`update_range`** covers part of a buffer without cutting a slice
  out of it, for a frame whose header and trailer are outside the
  check.
- **`step` carries no representation.**  One byte, `Int` in and `Int`
  out, so a target with no heap can hold the table as a constant and
  use the same function.
