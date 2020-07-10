# XYZ Image Specification
## Introduction
The XYZ image format (`.xyz`) is a custom image format used by RPG Maker 2000/2003 (RM2k/3).
This format is palette-based and stores three 8-bit color channels (red, green, and blue) for a total of 24 bits per pixel.

RM2k/3 uses this format used alongside more conventional formats, such as BMP and PNG.

## XYZ File Structure
XYZ files are stored in binary format.

The following table describes the overall structure of an XYZ file.

| Field     | Type                 | Description                                                  |
|:----------|:---------------------|:-------------------------------------------------------------|
| Signature | `U8[4]`              | The file's signature; this should always be "XYZ1" in ASCII. |
| Width     | `U16`                | The width of the image in pixels.                            |
| Height    | `U16`                | The height of the image in pixels.                           |
| Palette   | `RGB[256]`           | The image's color palette.                                   |
| PixelData | `U8[Width * Height]` | The image's pixel data.                                      |

**Note:** The `Palette` and `PixelData` fields are compressed using the [zlib](https://en.wikipedia.org/wiki/Zlib) compression algorithm;
all data after the first eight bytes must be decompressed before use. This may be accomplished by using the [zlib](https://www.zlib.net/) library.

Every value within the `PixelData` array is merely an index into the `Palette`.
That is, starting from the image's top-left and progressing left-to-right, top-to-bottom, every pixel is represented by an index into the `Palette` array.

Moreover, because the `Width` and `Height` fields are `U16`, the maximum size of an XYZ image is 65535 x 65535 pixels.
