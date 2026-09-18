# Changelog

Newest first.  Below `1.0.0` a breaking change bumps the **minor**
number and a compatible one the **patch**; see [Version numbers in the
Orbit package registry](https://novo-lang.org/docs/registry/semver.html).

## 0.1.5 — 2026-09-18

The documentation and comments in plain prose; no signature changed.

## 0.1.4 — 2026-09-08

- **The manifest names the layer.**  `layer = "core"` says that the
  public API requires no effects, and `novo pkg publish` checks the code
  against that.  No code changed.  The layers are described under Design
  in the [publishing guide](https://novo-lang.org/docs/publishing.html#design).
- **The sources in the canonical form** `novo fmt` prints today.  The
  change is spacing and alignment only, and no code changed.

## 0.1.3

The API reference generated from the code, with the examples in it run
as tests.  No code changed, and every check value is what 0.1.2
computed.

- **Every `pub` item documented under Go's rule**, in the comment block
  directly above the declaration, its first sentence the summary a
  reader meets before opening anything.  The methods of `impl Crc` carry
  their own.  `novo doc` turns that into
  [the package's page](https://novo-lang.org/packages/crc-nv).
- **Eight worked examples, and they run.**  Each constructor computes
  its own published check value over `123456789`, and the streaming form
  is shown reaching the same answer as the one-call form.  A fenced
  `novo` block in a documentation comment is compiled by `novo doc` and
  run by `novo test src/crc.nv`, so a check value that stopped being
  true is a failing test.
- **The README's algorithm table is gone.**  The parameters and the
  check values are on the generated page, where they are read out of the
  code that computes them.  The README keeps which format uses which
  algorithm, and why the misleading name is kept.

## 0.1.2

The Apache-2.0 text in the tarball, and the sources released from
`novolang/crc-nv`.  No code changed, and every signature and every byte
on the wire is what 0.1.1 shipped.

- **`LICENSE` ships with the package.**  It is on the publish
  allow-list, so the tarball carries the Apache-2.0 text rather than
  only naming it in the manifest.
- **The sources, CI and the release tags live in
  `novolang/crc-nv`** from this version on.

## 0.1.1

The four algorithms written with the operators, and the test suite out
of `src/`.  Every check value is the value it was and no signature
moved.

- **The table builders and the step are written with the operators.**
  Twenty-five `bits.*` calls become `&`, `^`, `<<` and `>>>`, and the
  reflected table's shift-only branch is `remainder >>>= 1`.  The
  reflected step is two lines a reader can check against the algorithm.
  `(state ^ byte) & 0xff` selects the table entry, and
  `self.table[index] ^ (state >>> 8)` shifts the rest of the state into
  it.  Every shift count that is an expression is given a name, because
  the shifts bind looser than `-` and the inline form reads wrong.
  `std.bits` is no longer imported.
- **The test module moved out of `src/`.**  A package's `src/` ships
  whole and a consumer compiles every module in it, so the suite is
  under `tests/` where it is not published.  Run it with
  `novo test tests/crc_tests.nv`.

## 0.1.0

The four algorithms: `crc16_ccitt_false`, `crc16_modbus`, `crc32` and
`crc32c`, and the `Crc` value's `start`, `step`, `update`,
`update_range`, `finish`, `checksum` and `verify`.

- **Four fully specified algorithms.**  Polynomial, initial state,
  reflection and final xor are all named, and each algorithm is asserted
  against the check value its catalogue entry publishes.
- **Streaming is the primitive.**  The running state is an `Int`, so a
  message that arrives in pieces gives the same answer as one that
  arrived whole.  `checksum` is the one-call form on top.
- **`update_range`** covers part of a buffer without cutting a slice out
  of it, for a frame whose header and trailer are outside the check.
- **`step` carries no representation.**  It takes one byte, with an
  `Int` in and an `Int` out, so a target with no heap can hold the table
  as a constant and call the same function.
