# Byte Order
All binary files (i.e. XYZ and Lcf files) are assumed to be using little-endian byte order:
the least significant byte is stored first, and the most significant byte is stored last.

For example, the 32-bit integer `0x456E7120` would be stored as `20 71 6E`.

# Data Types
The following are all of the data types that may be present within one of RM2k/3's custom file formats.

All types may be appended with `[n]` to denote a contiguous array of said type, where `n` denotes the number of elements.
For example, `U8[4]` denotes a contiguous array of four unsigned 8-bit integers.

## Basic Data Types
These are the basic data types that will be taken as primitive. The reader is assumed to already be familiar with these data types.

### Integer Types
Signed integers are represented in the usual two's complement format.

| Type   | Description                                                                |
|:-------|:---------------------------------------------------------------------------|
| `U8`   | An unsigned 8-bit integer.                                                 |
| `U16`  | An unsigned 16-bit integer.                                                |
| `U32`  | An unsigned 32-bit integer.                                                |
| `BOOL` | An unsigned 8-bit integer acting as a boolean (`0` is false; `1` is true). |
| `FLAG` | An unsigned 8-bit integer that acts as a flag/toggle.                      |
| `EINT` | A 7-bit encoded integer. See [Encoded Integers](#encoded-integers).        |

#### Flag Values
`FLAG` fields may have the following values.

| Type    | Value | Description                |
|:--------|:------|:---------------------------|
| Inherit | 0     | Inherit value from parent. |
| Allow   | 1     | Allow/enable flag.         |
| Forbid  | 2     | Forbid/disable flag.       |

#### Encoded Integers
Lcf files make extensive use of 7-bit encoded integers to cut down on file size.
These integers have a variable storage size, unlike the more common fixed-length formats.
Ideally, they will only take up the minimum number of bytes needed to encode their value.

For example, `123456` should only take up three bytes.

Only the lower seven bits of an encoded integer will contribute to its overall value;
the eighth (high) bit indicates whether or not the there is another byte that should be read, decoded, and added to the overall value.

Below is some pseudocode for reading and decoding these encoded integers into a 32-bit signed integer.

**Note:** the code below makes no assumption on the size of the `EINT` which may not be efficient if the exact size is known,
and the code does not perform any sort of error checking.

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
These are the usual IEEE floating-point types.

| Type  | Description                     |
|:------|:--------------------------------|
| `F32` | A 32-bit floating-point number. |
| `F64` | A 64-bit floating-point number. |

## Compound Data Types
Compound data types are built from the basic data types described above. These correspond to structures/records in many programming languages.

### STRING Type
The `STRING` type represents a length-prepended string of characters. These are used to store textual data.

When present within a tag of an Lcf file, the string's length will not be present. Instead, the tag's size field should
be used as the string's length.

When present within a text file, this type is simply a sequence of characters with no explicit length.
For an example, see the `GameTitle` field of the [configuration file format](config.md).

| Field  | Type         | Description                             |
|:-------|:-------------|:----------------------------------------|
| Length | `EINT`       | The length of the string in bytes.      |
| Data   | `U8[Length]` | The characters that make up the string. |

**Note:** Strings have no standard encoding, and the official Runtime performs little to no error checking;
the operating system's locale is used. (Japanese games will typically use [Shift JIS](https://en.wikipedia.org/wiki/Shift_JIS).)

This is lack of regularity is likely due to RM2k/3 originally being a Japanese-only engine that only received an English port years later;
Shift_JIS was assumed to be the encoding of choice. This has resulted in various different encodings being used across RM2k/3 games,
although UTF-8 has become dominant for modern games.

### LIST Type
The `LIST` type represents an ordered list of elements. Notation-wise, this will always be followed by another type to
indicate what type the elements are.

This is essentially an array with the caveat that, when stored in a binary file,
all elements are to be preceded by their 1-based index and followed by an [End Tag](common_tags.md#end-tag).

For example, suppose `L` is of type `LIST U8` with the elements 0, 25, 50, 75, and 100, respectively.
When stored in an Lcf file, `L` should be laid out as follows:

```text
1 0 0   2 25 0   3 50 0   4 75 0   5 100 0
```

Notice how the elements are stored in their usual way while being sandwiched between their 1-based index and an end tag (`0`).

### DATE Type
The `DATE` type represents a date and time.

This type corresponds to the `TDateTime` type found in Delphi and other Pascal variants.

| Field  | Type  | Description                                |
|:-------|:------|:-------------------------------------------|
| Date   | `F64` | The numeric encoding of the date and time. |

The `Date` field is stored such that the integral part is for days and the fractional part is for the time of day.

**Note:** Timezone and other considerations are ignored; `DATE` only stores raw day and time values.

### RGB Type
The `RGB` types represents a color with three color values: red, green, and blue.

| Field | Type | Description                    |
|:------|:-----|:-------------------------------|
| Red   | `U8` | The amount/intensity of red.   |
| Green | `U8` | The amount/intensity of green. |
| Blue  | `U8` | The amount/intensity of blue.  |
