# Byte Order
All binary files (i.e. XYZ and Lcf files) are assumed to be using little-endian byte order:
the least significant byte is stored first, and the most significant byte is stored last.

For example, the 32-bit integer `0x456E7120` would be stored as `20 71 6E`.

# Data Types
These are all of the data types that may be present within the external files of RPG Maker 2000/2003 (RM2k/3).

All types may be appended with `[n]` to denote a contiguous array of said type, where `n` indicates the number of elements.
For example, `U8[4]` denotes a contiguous array of four unsigned 8-bit integers.

## Basic Data Types
These are the basic data types that will be taken as primitive. The reader is assumed to already be familiar with these types.

### Integer Types
Signed integers are represented in the usual two's complement format.

| Type   | Description                                                                |
|:-------|:---------------------------------------------------------------------------|
| `U8`   | An unsigned 8-bit integer.                                                 |
| `U32`  | An unsigned 32-bit integer.                                                |
| `I32`  | A signed 32-bit integer.                                                   |
| `BOOL` | An unsigned 8-bit integer acting as a boolean (`0` is false; `1` is true). |
| `EINT` | A 7-bit encoded integer. See [Encoded Integers](#encoded-integers)         |

#### Encoded Integers
Lcf files make extensive use of 7-bit encoded integers to cut down on file size.
These integers vary in size, unlike the more common fixed-length formats,
and will only take up the minimum number of bytes needed to encode a particular value.

For example, `123456` will only take up three bytes.

Only the lower seven bits of an encoded integer will contribute to its overall value;
the eighth (high) bit is used to determine whether or not the there is another byte that should be read, decoded, and added to the overall value.

Below is some pseudocode for reading and decoding these encoded integers into an `I32` integer. Note, however, that the provided code
reads until done which may not be very efficient if the exact size of the integer in bytes is known.

```rust,ignore
i32 read_encoded_integer(reader) {
    i32 result = 0;

    while true {
        u8 next_byte = reader.read_u8();

        result <<= 7;
        result |= (next_byte & 0x7F) as i32;

        if (next_byte & 0x80) == 0 {
            break;
        }
    }

    return result;
}
```

### Floating-Point Types
These are the usual floating-point types; no further details necessary.

| Type  | Description                    |
|:------|:-------------------------------|
| `F32` | A 32-bit floating-point number. |
| `F64` | A 64-bit floating-point number. |

## Compound Data Types
These data types are built from the other data types and correspond to structures/records in many programming languages.

### STRING Type
The `STRING` type represents a length-prepended string of characters. These are used for all textual data.

When present within a tag of an Lcf file, the string's length will not be present. Instead, the tag's size field should
be used as the string's length. This further cuts down on the overall size of Lcf files.

When present within a text file, this type is simply a sequence of characters with no explicit length.
For an example, see the `GameTitle` field of the [configuration file](config.md).

**Note:** Strings have no standard encoding, and it is the runtime's duty to determine an appropriate encoding.
This is typically based on the operating system's current locale. (Japanese games will typically use [Shift JIS](https://en.wikipedia.org/wiki/Shift_JIS).)

This is likely due to RM2k/3 originally being a Japanese-only engine; Shift_JIS was assumed to be the encoding of choice.
This has resulted in various different encodings being used across RM2k/3 games, although UTF-8 has become dominant for modern games,
and the engine did get an official English port over a decade after release.

| Field  | Type         | Description                             |
|:-------|:-------------|:----------------------------------------|
| Length | `EINT`       | The length of the string in bytes.      |
| Data   | `U8[Length]` | The characters that make up the string. |

### LIST Type
The `LIST` type represents an ordered list of elements.
This type will always be followed by another type which indicates the type of elements being stored.

This is essentially an array with the caveat that, when stored in a binary file,
all elements are to be preceded by their 1-based index and followed by an [End Tag](common_tags.md#end-tag).

For example, suppose `L` is of type `LIST U8` with the elements 0, 25, 50, 75, and 100, respectively.
When stored in an Lcf file, `L` should be laid out as follows:

```text
1 0 0   2 25 0   3 50 0   4 75 0   5 100 0
```

Notice how the elements are stored in their usual way while being sandwiched between their 1-based index and an end tag (`0`).

### RGB Type
The `RGB` types represents a color with three color values: red, green, and blue.

| Field | Type | Description                    |
|:------|:-----|:-------------------------------|
| Red   | `U8` | The amount/intensity of red.   |
| Green | `U8` | The amount/intensity of green. |
| Blue  | `U8` | The amount/intensity of blue.  |
