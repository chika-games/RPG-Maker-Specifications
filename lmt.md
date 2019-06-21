# LCF Map Tree Specification (LMT)
| Key | Value |
| --- | --- |
| Version | _UNDER CONSTRUCTION_ |
| Status | Finalization, Speculation |
| License | [![Creative Commons License](https://i.creativecommons.org/l/by-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-sa/4.0/) |

## Table of Contents
* Table of Contents
* [Introduction](#introduction)
* [Data Types](#data-types)
    * [Basic Data Types](#basic-data-types)
        * [Variable-Size Integers](#variable-size-integers)
    * [Complex Data Types](#complex-data-types)
        * [String Values](#string-values)
* [LMT File Structure](#lmt-file-structure)
* [Document Changes](#document-changes)
* [Attribution](#attribution)

## Introduction
LCF Map Tree files are used to store map properties, map orderings, and game start information.

__Speculation:__ Any piece of information superscripted with a question mark (?) is speculation and should not be treated as fact.

This specification is part of a larger collection that can be found at the following link: [https://github.com/napen123/rpgmaker-asset-specs](https://github.com/napen123/rpgmaker-asset-specs)

## Data Types
This section describes the various data types that will be used throughout this document.

Types may be appended with `[n]` in order to denote a contiguous array of said type where `n` is the number of elements. For example, `U8[4]` denotes an array of four unsigned 8-bit integers.

### Basic Data Types
These types are considered fundamental as all other types are simply a combination of these.

| Type | Description |
| --- | --- |
| U8 | An unsigned 8-bit integer. |
| U32 | An unsigned 32-bit integer. |
| VINT | An integer of variable size. See [Variable-Size Integers](#variable-size-integers) for more information. |

#### Variable-Size Integers
LCF files predominantly use variable-size integers instead of fixed-length integers, such as 32-bit integers. Variable-size integers take up the least number of bytes necessary to encode a particular value; for example, the value `25` will be represented using a single byte whereas `123456` will take up two. These types of integers allow LMT files to be more compact and take up less disk space.

Below is some pseudocode for reading these integers:
```python
u32 read_variable_integer():
    u32 ret = 0
    u8 current_byte = read_u8()
    
    while (current_byte & 0x80) != 0:
        ret = (ret << 7) | (current_byte & 0x7F)
        current_byte = read_u8()
    
    return ret
```

### String Types
__Note:__ There is no guarantee as to what encoding a string's characters are in. Many Japanese games use the [Shift JIS](https://en.wikipedia.org/wiki/Shift_JIS) encoding though more modern games will use UTF-8. Discretion is advised.

#### `STRING`
The `STRING` type represent a length-prepended string of characters.

| Field | Type | Description |
| --- | --- | --- |
| Length | VINT | The number of characters that make up the string. |
| Characters | U8[`Length`] | The array of characters. |

#### `CHARS`
The `CHARS` type is only a string of characters; it is an alias for `U8[Length]` where `Length` is given somewhere else.

| Field | Type | Description |
| --- | --- | --- |
| Characters | U8[`Length`] | The array of characters where `Length` is specified separately. |

### Array Types
In addition to the standard `[n]` suffix, LMT files also utilize special 1D and 2D arrays.

__Note:__ Due to the way these array types are used, they may be best treated as [dictionaries](https://en.wikipedia.org/wiki/Associative_array) (hashmaps) or tag-based structures similar to a SWF file.

#### 1D Arrays
The `1D` _prefix_ denotes a null-terminated array. The following structure continuously repeats - in order - until a null `Index` (0) is encountered.

| Field | Type | Description |
| --- | --- | --- |
| Index | VINT | The index of the element; the "key" of a dictionary entry. If this is null (0), then no `Element` follows and the array is complete. |
| Element | TYPE | An element of the `TYPE` being prefixed as `1D`; the "value" of a dictionary entry. |

For example, `1D U8` denotes a null-terminated array of unsigned 8-bit integers.

#### 2D Arrays
The `2D` _prefix_ denotes a length-prepended array of `1D` arrays. Fields marked with * continuously repeat - in order - a specified number of times (`Length`).

| Field | Type | Description |
| --- | --- | --- |
| Length | VINT | The number of `Index` and `Element` fields to follow. |
| Index* | VINT | The index of the element; the "key" of a dictionary entry. |
| Element* | 1D TYPE | A `1D` array of the `TYPE` being prefixed as `2D`. |

For example, `2D U8` denotes a length-prepended array of `1D U8`.

## LMT File Structure
This section details the overall structure of an LMT file. None of the fields specified should be assumed to exist<sup>?</sup>. In such a case, the fields take on a specified default value (see each structure).

| Field | Type | Description |
| --- | --- | --- |
| Signature | STRING | The value of this field should always be "LcfMapTree". This field helps determine if a file is a valid LMT file. |
| MapInfos | 2D [Map Info](#map-info-structure) | A 2D array of map info structures. |
| MapOrder | [Map Order](#map-order-structure) | A map order structure. |
| ActiveNode | VINT | ??? |
| MapStart | 1D [Map Start](#map-start-structure) | A 1D array of map start structures. |

## Map Info Structure
Each Map Info structure contains properties for a particular map. All maps in a game should have a Map Info associated with it. None of the fields specified should be assumed to exist unless stated otherwise. In such a case, the fields take on a provided default value.

Historically, the first map (the root map) was used to specify the game's name. This was changed, however: a game's name is now stored in another file (RPG_RT.ini). Despite this, the root map is still stored and should be expected, but it shouldn't be treated like a playable map.

| Field | Type | Default Value | Description |
| --- | --- | --- | --- |
| Size | VINT | Always present | The total size of the following fields in bytes<sup>1</sup>. |

<sup>1</sup> This field can be used to skip Map Infos: simply read `Size` number of bytes.

## Document Changes

## Attribution
RPG Maker is property of Enterbrain, Inc. and Kadokawa Corporation.
