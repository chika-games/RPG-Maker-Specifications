# XYZ Image Specification
| Key | Value |
| --- | --- |
| Version | 2.0.0 |
| Status | Complete |
| License | [![Creative Commons License](https://i.creativecommons.org/l/by-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-sa/4.0/) |

## Table of Contents
* Table of Contents
* [Introduction](#introduction)
* [Data Types](#data-types)
  * [Basic Data Types](#basic-data-types)
  * [Complex Data Types](#complex-data-types)
    * [RGB Values](#rgb-values)
* [XYZ File Structure](#xyz-file-structure)
* [Document Changes](#document-changes)
* [Attribution](#attribution)

## Introduction
The XYZ file format is a custom image format used in RPG Maker 2000 and RPG Maker 2003. This simple format is palette based and stores three 8-bit color channels (red, green, and blue) for a total of 24 bits per pixel. The [DEFLATE](https://en.wikipedia.org/wiki/DEFLATE) compression algorithm is used to compress the file's palette and pixel data.

XYZ files typically have the extension `.xyz`.

## Data Types
This section describes the various data types that will be used throughout this document. Little-endian is assumed.

Types may be appended with `[n]` in order to denote a contiguous array of said type where n is the number of elements. For example, `U8[4]` denotes an array of four unsigned 8-bit integers.

### Basic Data Types
These types are considered _primitive_ as all other data structures are built using these fundamental types.

| Type | Description |
| --- | --- |
| U8 | An unsigned 8-bit integer. |
| U16 | An unsigned 16-bit integer. |

### Complex Data Types
These types are built using a combination of [Basic Data Types](#basic-data-types).

#### RGB Values
`RGB` values represent a color through three channels: red, green, and blue.

| Field | Type | Description |
| --- | --- | --- |
| Red | U8 | The amount of red in the color. |
| Green | U8 | The amount of green in the color. |
| Blue | U8 | The amount of blue in the color. |

## XYZ File Structure
This section details the structure of an XYZ file.

| Field | Type | Description |
| --- | --- | --- |
| Signature | U8[4] | This field is used to determine if a file claims to be a valid XYZ file. The value of this field should always be "XYZ1". |
| Width | U16 | The width of the image in pixels. |
| Height | U16 | The height of the image in pixels. |
| Palette<sup>1</sup> | RGB[256] | The image's palette; this contains all of the colors that will be present in the image. |
| PixelData<sup>1</sup> | U8[`Width * Height`] | The image's pixel data; these are indexes into the image's `Palette` where each index represents a pixel. |

<sup>1</sup> All of the data after the height parameter (after the first 8 bytes) is compressed using the [DEFLATE](https://en.wikipedia.org/wiki/DEFLATE) compression algorithm. Be the rest of the file is uncompressed before reading these fields. This can be achieved by using zlib or a similar library.

## Document Changes
### Version 2.0.0
Rewrote the document from scratch.

### Version 1.0.0
First version of the document released.

## Attribution
RPG Maker is property of Enterbrain, Inc. and Kadokawa Corporation.
