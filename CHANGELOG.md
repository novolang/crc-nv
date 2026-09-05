# Changelog

Newest first.  Below `1.0.0` a breaking change bumps the **minor**
number and a compatible one the **patch**; see [Version numbers in the
Orbit package registry](https://novo-lang.org/docs/registry/semver.html).

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
