# XYZ Image Specification
## Background
XYZ files are what RPG Maker 2000/2003 use to store image data.

| Key | Value |
| --- | --- |
| Version | 1.0.0 |
| Status | Complete |
| License | [![Creative Commons License](https://i.creativecommons.org/l/by-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-sa/4.0/) |

## Overview
XYZ is a palette-based image format with 24 bits per channel (red, green, blue). The palette can store up to 256 distinct colors, and the pixel data is compressed using the [DEFLATE](https://en.wikipedia.org/wiki/DEFLATE) compression algorithm.

Overall, the XYZ format is very compact and has a simple specification. An example of a piece of code that reads XYZ files (among other things) can be found [here](https://github.com/napen123/xyz2png/blob/master/src/main.rs).

## Type Notation
This is a table of notations used within this document to denote various types of values:

| Notation | Description |
| --- | --- |
| U8 | An unsigned 8-bit integer. |
| U16 | An unsigned 16-bit integer. |
| RGB | Three unsigned 8-bit integers representing the color channels red, green, and blue, respectively. This is equivalent to `U8[3]`. |

Types may be appended with `[n]`, where n is a positive integer, in order to denote a contiguous chunk of said type. For example, `U8[4]` represents four contiguous unsigned 8-bit integers.

## Memory Layout
The following describes the overall structure of an XYZ file. The contents are listed in the order they appear in the file.

| Name | Type | Description |
| --- | --- | --- |
| `Header` | U8[4] | The header is always `XYZ1` and should be used to determine if a file represents an XYZ image. |
| `Width` | U16 | The width of the image in pixels. |
| `Height` | U16 | The height of the image in pixels. |
| `Palette`<sup>1</sup> | RGB[256] | The image's palette; this contains all of the colors that will be present in the image. |
| `Pixel Data`<sup>1</sup> | U8[`Width` * `Height`] | Indexes into the image `Palette`. Each index represents a pixel within the image; one index for every pixel. |

<sup>1</sup> All of the data after the height parameter (after the first 8 bytes) is compressed using the [DEFLATE](https://en.wikipedia.org/wiki/DEFLATE) compression algorithm. Be sure you uncompress the rest of the file! This can be achieved by using zlib or something similar.
