---
layout: default
---

# XYZ Image Specification
## Introduction
The XYZ image format (*.xyz*) is a custom image format used by RPG Maker 2000/2003.
This format is palette-based and stores three 8-bit color channels (red, green, and blue) for a total of 24 bits per pixel.
This results is a compact format that's easy to read and write.

This format is used alongside the more conventional BMP and PNG image formats.

## Data Types
Little-endian byte order is assumed for all numerical types;
the least significant byte is stored first, and the most significant byte is stored last.

Types may be appended with `[n]` to denote a contiguous array of the type in question, where *n* indicates the number of elements.
For example, `U8[4]` denotes a contiguous array of four unsigned 8-bit integers.

### Basic Data Types
The following tables list of the basic data types that will be used within this specification.

| Type   | Description                 |
|:-------|:----------------------------|
| `U8`   | An unsigned 8-bit integer.  |
| `U16`  | An unsigned 16-bit integer. |

### Compound Data Types
Compound data types consist of a combination of basic data types.
These correspond to structures and/or records in most programming languages.

#### RGB Type
The `RGB` types represents a color with three color values: red, green, and blue.

| Field | Type | Description                    |
|:------|:-----|:-------------------------------|
| Red   | `U8` | The amount/intensity of red.   |
| Green | `U8` | The amount/intensity of green. |
| Blue  | `U8` | The amount/intensity of blue.  |

## XYZ File Structure
XYZ files are stored in binary format.

The following table describes the overall structure of an XYZ file.

| Field     | Type                | Description                                           |
|:----------|:--------------------|:------------------------------------------------------|
| Signature | `U8[4]`             | The file's signature; this should always be "XYZ1".   |
| Width     | `U16`               | The width of the image in pixels.                     |
| Height    | `U16`               | The height of the image in pixels.                    |
| Palette   | `RGB[256]`          | The image's color palette.                            |
| PixelData | `U8[Width * Height]`| The image's pixel data.                               |

**Note:** The `Palette` and `PixelData` fields are compressed using the [DEFLATE](https://en.wikipedia.org/wiki/DEFLATE) compression algorithm.
All data after the first eight bytes must be decompressed before use. This may be accomplished by using the [zlib](https://www.zlib.net/) library.

Every value within the `PixelData` array is merely an index into `Palette`.
That is, starting from the image's top-left and progressing right-down, every pixel is represented by an index into the `Palette`.

Moreover, due to using `U16` for sizes, the maximum size of an XYZ image is 65535 x 65535 pixels.
