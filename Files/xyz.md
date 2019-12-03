# XYZ Image Specification (XYZ)
## Table of Contents
* Table of Contents
* [Introduction](#introduction)
* [Data Types](#data-types)
  * [Basic Data Types](#basic-data-types)
  * [Complex Data Types](#complex-data-types)
    * [RGB Type](#rgb-type)
* [XYZ File Structure](#xyz-file-structure)
* [Document Changes](#document-changes)
* [Legal Information](#legal-information)

## Introduction
The XYZ file format (`.xyz`) is the custom image format used in RPG Maker 2000 and 2003. This format is palette-based and stores three 8-bit color channels (red, green, and blue) for a total of 24 bits per pixel. This results is a design that is simple to read and write.

The [DEFLATE](https://en.wikipedia.org/wiki/DEFLATE) (ZIP) compression algorithm is used to compress image palettes and pixel data.

This specification is part of a larger collection that can be found at the following URL: [https://github.com/chika-games/RPG-Maker-Specifications](https://github.com/chika-games/RPG-Maker-Specifications)

## Data Types
This section describes the various data types that will be used throughout this specification.

Little-endian byte order is assumed for all types; the least significant byte is stored first, and the most significant byte is stored last.

Types may be appended with `[n]` in order to denote a contiguous array of the type with `n` being the number of elements. For example, `U8[4]` denotes an array of four unsigned 8-bit integers.

### Basic Data Types
These types are considered _primitive_ as all other data types are constructed using these.

| Type | Description |
| --- | --- |
| U8 | An unsigned 8-bit integer. |
| U16 | An unsigned 16-bit integer. |

### Complex Data Types
These types are built using a combination of [Basic Data Types](#basic-data-types).

#### RGB Type
The `RGB` type represents a color with three channels: red, green, and blue.

| Field | Type | Description |
| --- | --- | --- |
| Red | U8 | The amount of red in the color. |
| Green | U8 | The amount of green in the color. |
| Blue | U8 | The amount of blue in the color. |

## XYZ File Structure
This section details the overall structure of an XYZ file.

| Field | Type | Description |
| --- | --- | --- |
| Signature | U8[4] | This field is the file's signature. The value of this field should always be "XYZ1". |
| Width | U16 | The width of the image in pixels. |
| Height | U16 | The height of the image in pixels. |
| Palette<sup>1</sup> | RGB[256] | The image's palette; this contains all of the colors that may be present in the image. |
| PixelData<sup>1</sup> | U8[`Width * Height`] | The image's pixel data; these are indexes into the image's `Palette` where each index represents a pixel. |

<sup>1</sup> All of the fields after the `Height` parameter (after the first 8 bytes) is compressed using the [DEFLATE](https://en.wikipedia.org/wiki/DEFLATE) compression algorithm; all fields before this are uncompressed. Libraries such as zlib may be used to decompress this data.

## Legal Information
This document is provided under the [CC BY-SA 4.0](http://creativecommons.org/licenses/by-sa/4.0/) license. The RPG Maker trademark and copyright are property of Enterbrain, Inc. and Kadokawa Corporation.

All rights belong to their respective owners.
