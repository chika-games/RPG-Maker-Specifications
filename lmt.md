# LCF Map Tree Specification (LMT)
## Background

| Key | Value |
| --- | --- |
| Version | _UNDER CONSTRUCTION_ |
| Status | _Research/Speculation_ |
| License | [![Creative Commons License](https://i.creativecommons.org/l/by-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-sa/4.0/) |

## Overview

## Type Notation
This is a table of notations used within this document to denote various types of values:

| Notation | Description |
| --- | --- |
| STRING | An `EINT` `length` value followed by a contiguous chunk of `length`-many unsigned 8-bit integers. (I.e. a length followed by characters.) |
| CHUNK | Equivalent to `STRING`: an `EINT` length followed by unsigned 8-bit values. This represents an arbitrary piece of data unless otherwise noted. |
| EINT | An encoded variable-length signed integer. See [Encoded Integers](#encoded-integers) for more information. |
| BOOL | A boolean value. When represented using an integer, zero is _false_ and nonzero is _true_.

## Memory Layout
The following describes the overall structure of an LMT file from top to bottom:

| Name | Type | Description |
| --- | --- | --- |
| `Header` | STRING | The header always has a length of 10 and a value of `LcfMapTree`. This is used to determine if a file represents an LMT file. |
| `Info Count` | EINT | The number of [Map Information] structures to follow. |
| `Map Info` | [Map Information] [Info Count] | An array of [Map Information] structures. |
| `Order Count` | EINT | The number of map orderings to follow. |
| `Map Order` | EINT[`Order Count`] | An array of map IDs in sequential order. For example, the first map of a game is the first element in the array. |
| `Active Node` | EINT |  |
| `Start` | [Map Start](#map-start) | A structure containing information about starting maps and positions. |

Types may be appended with `[n]`, where n is a positive integer, in order to denote a contiguous array of said type. For example, `U8[4]` represents an array of four unsigned 8-bit integers.

[Map Information]: #map-information

### Encoded Integers
LCF files store integers in an encoded format. Below is pseudo-code for reading and decoding these integers:

```python
int read_encoded_integer(reader):
    var ret = 0
    
    while true:
        var b = reader.read_u8()
        
        ret <<= 7
        ret |= b & 0x7F
        
        if (b & 0x80) == 0:
            break
    
    return ret
```

(Based on code from [gabien-app-r48](https://github.com/20kdc/gabien-app-r48).)

### Map Information
Map information is layed out as an `EINT` ID, followed by an `EINT` length, then either a `CHUNK[length]` or `STRING[length]`. In most cases, `length` will be 1.

There is no guarantee that the following chunks will be present as they are sometimes omitted if redundant, but the order in which they're layed out will always be the same. If a chunk is not present, then it is assumed to have a listed default value.

| Name | Type | Description |
| --- | --- | --- |
| ID | EINT | The ID of one of the below items. |
| length | EINT | The number of items in the data array that follows this field. Usually 1. |
| data | CHUNK[`length`] or STRING[`length`] | See the table below in order to interpret this data. |

| Name | ID (Hex) | Type and `length` | Default Value | Description |
| --- | --- | --- | --- | --- |
| Game Name | 0 | STRING[1] | An empty string. | Traditionally used to give the game's name. This should be ignored, however, as game names are specified in an accompanying INI file (i.e. `RPG_RT.ini`). |
| Map Name | 1 | STRING[1] | An empty string. | The name of the map being described. |
| Parent ID | 2 | CHUNK[1] | 0 | Either the ID of a parent map or 0. This should be treated like an unsigned integer. |
| Indentation | 3 | CHUNK[1] | 0 | This should be treated like an unsigned integer. |
| Map Type | 4 | CHUNK[1] | 0 | The type of map being described. See [Map Types](#map-types) for a list of special values. This should be treated like an unsigned integer. |
| Edit Position X | 5 | CHUNK[1] | 0 | For editor use. This should be treated like a signed integer. |
| Edit Position Y | 6 | CHUNK[1] | 0 | For editor use. This should be treated like a signed integer. |
| Edit Expanded | 7 | CHUNK[1] | false (0) | For editor use. This should be treated like a `BOOL`. |
| Music Type | B | CHUNK[1] | 0 | How music should be played in the map. See [Music Types](#music-types) for a list of special values. This should be treated like an unsigned integer. |

#### Map Types
| Name | Value |
| --- | --- |
| Root | 0 |
| Map | 1 |
| Area | 2 |

#### Music Types
| Name | Value |
| --- | --- |
| Inherit | 0 |
| Event | 1 |
| Specified | 2 |

### Map Start
This contains information about a game's starting maps and positions. Note: not all of the following may be present in an LMT file. If any of the following are absent, then they should be assumed to have a value of `0`.

| Name | Type | Description |
| --- | --- | --- |
| `Player Map` | EINT | The ID of the map to start the game in. This isn't the main menu. |
| `Player X` | EINT | The x-position in the `Player Map` to start the player at. |
| `Player Y` | EINT | The y-position in the `Player Map` to start the player at. |
| `Boat Map` | EINT | The ID of the boat map. |
| `Boat X` | EINT | The x-position in the `Boat Map` to start the player at. |
| `Boat Y` | EINT | The y-position in the `Boat Map` to start the player at. |
| `Ship Map` | EINT | The ID of the ship map. |
| `Ship X` | EINT | The x-position in the `Ship Map` to start the player at. |
| `Ship Y` | EINT | The y-position in the `Ship Map` to start the player at. |
| `Airship Map` | EINT | The ID of the airship map. |
| `Airship X` | EINT | The x-position in the `Airship Map` to start the player at. |
| `Airship Y` | EINT | The y-position in the `Airship Map` to start the player at. |

## Attribution
In addition to personal research and digging, information within this document is also based on the [gabien-app-r48](https://github.com/20kdc/gabien-app-r48) repository.
