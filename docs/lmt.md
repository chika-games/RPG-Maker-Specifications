---
layout: default
---

# LCF Map Tree File Specification
## Introduction
LCF Map Tree (LMT) files are used to store map properties, game start information, and map orderings for RPG Maker 2000/2003 games.

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
the eighth (high) bit is used to determine whether or not the there is another byte that should be read and decoded.

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

#### STRING Type
The `STRING` type represents a length-prepended string of characters. These are used for all textual data.

**Note:** Strings have no standard encoding. It is up to the runtime's discretion as to which encoding to use;
this is typically based on the operating system's current locale. Japanese games will typically use [Shift JIS](https://en.wikipedia.org/wiki/Shift_JIS).

| Field  | Type         | Description                             |
|:-------|:-------------|:----------------------------------------|
| Length | `EINT`       | The length of the string in bytes.      |
| Chars  | `U8[Length]` | The characters that make up the string. |

## LCF Map Tree File Structure
LMT files are stored in binary format.

The following table describes the overall structure of an LMT file.

| Field         | Type                     | Description                                               |
|:--------------|:-------------------------|:----------------------------------------------------------|
| Signature     | `STRING`                 | The file's signature; this should always be "LcfMapTree". |
| MapInfoCount  | `EINT`                   | The number of map info structures present.                |
| MapInfos      | `Map Info[MapInfoCount]` | An array of map info structures.                          |
| MapOrderCount | `EINT`                   | The number of map orderings present.                      |
| MapOrders     | `EINT[MapOrderCount]`    | The hierarchical orderings of a game's maps.              |
| ActiveNode    | `EINT`                   | The ID of the last active/edited map.                     |
| MapStart      | `Map Start`              | Game start information, such as starting positions.       |

`ActiveNode` is used by the RPG Maker 2000/2003 editors to keep track of the last edited map.
This allows to, upon launching, open the last edited map for easy access.

### Map Hierarchy

## Map Info Structure

## Map Start Structure

## Tags

### Map Info Tags

### Map Start Tags

### Music Tags

### Encounter Tags
