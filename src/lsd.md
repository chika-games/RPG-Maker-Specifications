---
layout: default
---

# LCF Save Data File Specification
## Introduction
LCF Save Data (LSD) files are used to store save data for RPG Maker 2000/2003 (RM2k/3) games.
LSD files use a tag-based format similar to the one used in e.g. Adobe SWF files. This allows for easy reading and writing.

These files are typically stored within the same directory as the game's executable file (`RPG_RT.exe`).

## Data Types
LSD files use little-endian byte ordering; the least significant byte is stored first, and the most significant byte is stored last.

Types may be appended with `[n]` to denote a contiguous array of said type, where `n` indicates the number of elements.
For example, `U8[4]` denotes a contiguous array of four unsigned 8-bit integers.

### Basic Data Types
The following table lists all of the basic data types that are taken as primitive and assumed to be understood herein.

| Type   | Description                     |
|:-------|:--------------------------------|
| `U8`   | An unsigned 8-bit integer.      |
| `U32`  | An unsigned 32-bit integer.     |
| `F64`  | A 64-bit floating-point number. |
| `EINT` | A 7-bit encoded integer.        |

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

**Note:** Strings have no standard encoding. It is the runtime's duty to determine an appropriate encoding.
This is typically based on the operating system's current locale. Japanese games will typically use [Shift JIS](https://en.wikipedia.org/wiki/Shift_JIS).

| Field  | Type         | Description                             |
|:-------|:-------------|:----------------------------------------|
| Length | `EINT`       | The length of the string in bytes.      |
| Data   | `U8[Length]` | The characters that make up the string. |

#### LIST Type
The `LIST` type represents an ordered list of elements.
This type will always be followed by another type which indicates the type of elements being stored.

This is essentially an array where all elements are preceded by their 1-based index and followed by an [End Tag](#end-tag).

For example, suppose `L` is of type `LIST U8` with the elements 0, 25, 50, 75, and 100. When stored in an LSD file, `L` would be laid out as follows:

```
1 0 0   2 25 0   3 50 0   4 75 0   5 100 0
```

Notice how the index of each element is stored directly in front of the element's data,
and that the [End Tag](#end-tag) is stored as 0.

#### DATE Type
The `DATE` type represents a date and time.

This type corresponds to the TDateTime found in Delphi and many other Pascal variants.

| Field  | Type  | Description                                |
|:-------|:------|:-------------------------------------------|
| Date   | `F64` | The numeric encoding of the date and time. |

The `Date` field is stored such that the integral part is for days and the fractional part is for the time.

**Note:** Timezone and other considerations are ignored; `DATE` only stores raw day and time values.

## LCF Save Data File Structure
LMT files are stored in binary format.

The following table describes the overall structure of an LSD file.

| Field         | Type                                            | Description                                                |
|:--------------|:------------------------------------------------|:-----------------------------------------------------------|
| Signature     | `STRING`                                        | The file's signature; this should always be "LcfSaveData". |
| Timestamp     | `DATE`                                          | The total playtime.                                        |
| HeroName      | `STRING` | An array of map info structures.                           |
| HeroLevel     | `EINT`                                          | The number of map orderings present.                       |
| HeroHP        | `EINT[MapOrderCount]`                           | The hierarchical orderings of a game's maps.               |
| FaceName1     | `EINT`                                          | The ID of the last saved/edited map.                       |
| FaceID1       | [Map Start](#map-start-structure)               | Game start information, such as starting positions.        |
| FaceName2     | `EINT`                                          | The ID of the last saved/edited map.                       |
| FaceID2       | [Map Start](#map-start-structure)               | Game start information, such as starting positions.        |
| FaceName3     | `EINT`                                          | The ID of the last saved/edited map.                       |
| FaceID3       | [Map Start](#map-start-structure)               | Game start information, such as starting positions.        |
| FaceName4     | `EINT`                                          | The ID of the last saved/edited map.                       |
| FaceID4       | [Map Start](#map-start-structure)               | Game start information, such as starting positions.        |
