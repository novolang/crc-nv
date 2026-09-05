# crc-nv

Cyclic redundancy checks, table driven. Four algorithms, each fully
specified — polynomial, initial state, reflection, final xor — so a
value computed here is the value every other implementation computes.
A CRC costs a table lookup and an xor per byte and catches every burst
of errors shorter than its width, which is why every lossy link carries
one.

```novo
use crc

fn main() [io]
    let algo = crc.crc32()
    println("${algo.checksum(bytes.from_str("123456789"))}")   // 3421780262
```

```
novo pkg add crc-nv
```

## The four algorithms

| Constructor | Polynomial | Init | Reflected | Final xor | Check |
|---|---|---|---|---|---|
| `crc.crc16_ccitt_false()` | `0x1021` | `0xffff` | no | — | `0x29b1` |
| `crc.crc16_modbus()` | `0x8005` | `0xffff` | yes | — | `0x4b37` |
| `crc.crc32()` | `0x04c11db7` | `0xffffffff` | yes | `0xffffffff` | `0xcbf43926` |
| `crc.crc32c()` | `0x1edc6f41` | `0xffffffff` | yes | `0xffffffff` | `0xe3069283` |

The **check** column is the algorithm's result over the nine ASCII
bytes `123456789`. Every CRC catalogue publishes one, and it is the
number to compare against when a value here disagrees with a value from
somewhere else — a mismatch is almost always the wrong variant rather
than a wrong implementation.

`CRC-32` is the one in zlib, gzip, PNG, ZIP and Ethernet. `CRC-32C`
uses the Castagnoli polynomial and is what iSCSI, SCTP, ext4 and Btrfs
carry. `CRC-16/MODBUS` is the frame check of Modbus RTU.
`CRC-16/CCITT-FALSE` is the name the catalogue gives the variant widely
called "CRC-16-CCITT" and which is not the one CCITT specified; the
misleading name is kept because it is the one a reader arrives holding.

## What it gives you

| Method | |
|---|---|
| `algo.checksum(src: Bytes) -> Int` | the check value of a whole buffer, in one call |
| `algo.verify(src: Bytes, expected: Int) -> Bool` | the receiver's question, worded as one |
| `algo.start() -> Int` | the state a fresh message begins from |
| `algo.update(state, src: Bytes) -> Int` | every byte of `src` into a running state |
| `algo.update_range(state, src, off, count) -> Int` | `count` bytes from `off`, so a header need not be sliced off |
| `algo.step(state, byte: Int) -> Int` | one byte — the whole algorithm, on `Int` alone |
| `algo.finish(state) -> Int` | the check value a running state has reached |

## Streaming

A message that arrives in pieces must produce the same check as one
that arrived whole, so `update` takes and returns the running state and
`finish` applies the final xor exactly once:

```novo
use crc

fn frame_check(header: Bytes, body: Bytes) -> Int
    let algo = crc.crc32()
    var state = algo.start()
    state = algo.update(state, header)
    state = algo.update(state, body)
    algo.finish(state)
```

`step` is the same thing one byte at a time — the shape a receiver
takes when the bytes come off a UART as they arrive.

## What it costs

**Build the algorithm once.** `crc.crc32()` computes a 256-entry table;
building one per message would cost more than the message. Hold the
`Crc` value and reuse it.

Checksumming is one table lookup, one xor and one shift per byte, and
allocates nothing — the running state is an `Int` and the buffer is
read, never copied. `update_range` exists so that a frame whose header
and trailer are outside the check needs no slice cut for them.

`step` takes and returns an `Int` and touches no buffer, so a target
with no heap can hold the table as a constant and use the same
function. It is the whole algorithm, in the reflected case one line:

```novo norun:fragment
fn step(self, state: Int, byte: Int) -> Int
    let index = (state ^ byte) & 0xff
    self.table[index] ^ (state >>> 8)
```

## What it does not do

No CRC-8, no CRC-64, and no custom polynomial: the four here are the
ones a wire format in this ecosystem actually names, and an
under-specified fifth is worse than none. No hardware acceleration —
`CRC-32C` has a processor instruction on x86-64 and AArch64 and this
package does not reach for it, so the cost is the table on every
target.

A CRC is an error check, not a signature. It detects accidental
corruption and does nothing at all against a change someone made on
purpose.

## Tests

```
novo test tests/crc_tests.nv
```

The published check values for all four algorithms, the empty message,
a streaming split asserted equal to the one-call answer, and the
single-flipped-bit sensitivity a CRC exists for.
