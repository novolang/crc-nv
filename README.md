# crc-nv

A cyclic redundancy check (CRC) is the remainder left when a message,
read as one long binary polynomial, is divided by a fixed generator
polynomial. It costs one table lookup and one exclusive-or per byte,
and it detects every burst of errors shorter than its own width. That
is why a lossy link carries one: a UART, a radio, a disk sector. This
package brings four named CRC algorithms to novo-lang, taken from the
[catalogue of parametrised CRC
algorithms](https://reveng.sourceforge.io/crc-catalogue/all.htm).

## What a cyclic redundancy check is

A CRC algorithm is named by four parameters. Two programs agree on a
number only when all four agree.

The **polynomial** is the divisor. It is written as the coefficients of
a binary polynomial, one bit each, with the highest coefficient left
out because it is always 1. The **initial state** is the value the
remainder starts from, before the first byte of the message is folded
in. **Reflection** says which end of each byte enters the division
first. A reflected algorithm feeds the least significant bit first and
shifts its state right. A non-reflected one feeds the most significant
bit first and shifts left. The **final xor** is the value
exclusive-ored into the remainder once the last byte is in.

The **width** is how many bits the result has. It is 16 bits for the
two 16-bit algorithms here and 32 bits for the other two. The width is
also what bounds the check. A burst of errors shorter than the width is
always detected.

A catalogue entry also publishes a **check value**. That is the
algorithm's result over the nine ASCII bytes `123456789`. The check
value is what to compare against when a value computed here disagrees
with a value from another program. A disagreement is almost always the
wrong variant rather than a wrong implementation, and the check value
says which variant each side is running.

## Install

```
novo pkg add crc-nv
```

## Example

```novo
use crc
use std.bytes

fn main() [io]
    // Build the algorithm once. The constructor computes a 256-entry
    // table.
    let algo = crc.crc32()
    // The check value of CRC-32, over the nine bytes the catalogue
    // quotes it for.
    println("${algo.checksum(bytes.from_str("123456789"))}")   // 3421780262
```

Build and test with `novo pkg build` and `novo test tests/crc_tests.nv`.

## What the package contains

| Module | Contents |
| --- | --- |
| `crc` | The `Crc` value, the four constructors that build one, and the calls a caller makes on it: `start`, `step`, `update`, `update_range`, `finish`, `checksum` and `verify`. |

The API reference is on
[the package's page](https://novo-lang.org/packages/crc-nv). `novo doc`
generates it from these sources: every `pub` declaration with its
signature, its effect row and the comment block written above it. Each
constructor's comment names its algorithm's four parameters and its
published check value, and an example beside it computes that value.

## How to choose an entry point

The four algorithms, and the formats that name them:

| Constructor | Where the algorithm is used |
| --- | --- |
| `crc32` | zlib, gzip, PNG, ZIP and Ethernet. This is the one to reach for when a format says only "CRC-32". |
| `crc32c` | iSCSI, SCTP, ext4 and Btrfs. It uses the Castagnoli polynomial, which RFC 3309 specifies for SCTP. |
| `crc16_modbus` | The frame check of Modbus RTU, in the Modbus over Serial Line specification. |
| `crc16_ccitt_false` | The 16-bit variant widely called "CRC-16-CCITT". It is not the algorithm CCITT specified. The catalogue's name for it is kept here, because it is the name a reader arrives holding. |

The calls, and who uses which:

**`checksum` and `verify` take the whole buffer.** `checksum` answers
the check value and `verify` answers whether the buffer carries a check
value the caller already has. This is the pair for a program that holds
all the bytes at once.

**`start`, `update` and `finish` take the message in pieces.** `start`
answers the state a message begins from, `update` folds a chunk into a
state and answers the new one, and `finish` turns a state into a check
value. This is the form for a message that arrives a chunk at a time.

**`update_range` covers part of a buffer.** It folds `count` bytes from
an offset, so a frame whose header and trailer are outside the check
needs no slice cut for them.

**`step` takes one byte.** It takes an `Int` and answers an `Int`, and
it reads no buffer at all. This is the call a receiver makes when the
bytes come off a UART as they arrive, and the one a target with no heap
allocator uses with the table held as a constant.

## The rules a user needs

1. **Build the algorithm once and keep the value.** A constructor
   computes a 256-entry table. Building one per message costs more than
   the message.
2. **A `Crc` value holds no running state.** One value serves every
   message and every thread. The running state is an `Int` the caller
   passes through.
3. **A running state is not a check value.** The final xor has not been
   applied to what `start` and `update` answer, so nothing should
   compare against one. `finish` applies the final xor, once.
4. **`finish` is called once per message.** Applying the final xor
   twice undoes it.
5. **`update_range` trusts the range it is given.** A byte outside the
   buffer reads as 0 rather than stopping the program, because a frame
   length is input.
6. **Modbus RTU sends the two check bytes low byte first.** That is the
   framing's business rather than this package's. `crc16_modbus`
   answers the number.
7. **A CRC is an error check and not a signature.** It detects
   accidental corruption. It does nothing at all against a change
   someone made on purpose.

## What is not included

- **CRC-8 and CRC-64.** The four here are the algorithms a wire format
  in this ecosystem names. `std.codec.crc8` is a CRC-8.
- **A custom polynomial.** A caller cannot pass one in, so every
  algorithm this package computes is one a catalogue entry specifies
  and a check value pins.
- **Hardware acceleration.** CRC-32C has a processor instruction on
  x86-64 and on AArch64, and this package does not reach for it. The
  cost is the table on every target.
- **A constant-time comparison.** `verify` is `checksum(src) ==
  expected` and nothing more, because a CRC is not a signature.
- **Any input or output.** Every function here is arithmetic over bytes
  the caller already holds, which is why the layer is `core` and no
  function declares an effect.

## Related packages

- `std.codec` in the standard library has `crc8`, `crc16_ccitt` and
  `crc32`. Each is an extern over a host C symbol, so each is one call
  and already linked on a host. There is no CRC-32C and no
  CRC-16/MODBUS among them, no streaming form, no range form, and the
  externs are available under the LLVM backend rather than on every
  target.

## Tests

```
novo test tests/crc_tests.nv
```

The suite asserts the published check value of all four algorithms, the
result over an empty message, a streaming split and a byte-at-a-time
walk that both agree with the one-call answer, `update_range` against a
framed message, that a single flipped bit changes the answer, that
`verify` answers both ways, that the reflected and non-reflected 16-bit
algorithms disagree, and that no result leaves its own width.

The check values come from the catalogue of parametrised CRC
algorithms. Every example in a documentation comment is compiled by
`novo doc` and run by `novo test`, so a check value that stopped being
true is a failing test.

## Licence

Apache-2.0. See `LICENSE`.

<!-- docs/writing-a-readme.md is the style guide for this page. -->
