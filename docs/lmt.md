---
layout: default
---

# LCF Map Tree File Specification
## Introduction
LCF Map Tree files are used to store map properties, game start information, and map orderings for RPG Maker 2000/2003 games.

## Data Types
Little-endian byte order is assumed for all numerical types;
the least significant byte is stored first, and the most significant byte is stored last.

Types may be appended with `[n]` to denote a contiguous array of the type in question, where `n` indicates the number of elements.
For example, `U8[4]` denotes a contiguous array of four unsigned 8-bit integers.

### Basic Data Types
The following table lists all of the basic data types that will be used within this specification.

| Type   | Description                 |
|:-------|:----------------------------|
| `U8`   | An unsigned 8-bit integer.  |
| `U32`  | An unsigned 32-bit integer. |
| `EINT` | A 7-bit encoded integer.    |

#### Encoded Integers
LCF files make extensive use of 7-bit encoded integers instead of the more common fixed-length formats.
These will typically only use the minimum number of bytes needed to encode a particular value.
For example, _123456_ will be encoded as three bytes. This helps reduce the overall size of LCF files.

As the name implies, only the lower seven bits of an encoded integer will contribute to its overall value;
the eighth (high) bit is used to determine whether or not the there another byte that follows the current one in memory.

Below is some pseudocode for reading and decoding these encoded integers into a fixed-length format (`U32`):

```c
u32 read_variable_integer(reader) {
    u32 ret = 0;

    while true {
        u8 byte = reader.read_u8();

        ret <<= 7;
        ret |= byte & 0x7F;

        if (byte & 0x80) == 0 {
            break;
        }
    }

    return ret;
}
```

### Compound Data Types
Compound data types consist of a combination of basic data types.
These correspond to structures and/or records in most programming languages.
