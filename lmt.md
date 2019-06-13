# LCF Map Tree Specification (LMT)
## Background

| Key | Value |
| --- | --- |
| Version | _UNDER CONSTRUCTION_ |
| Status | _Finalization & Speculation_ |
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
* [Map Info Structure](#map-info-structure)
* [Map Start Structure](#map-start-structure)
* [Tags](#tags)
* [Attribution](#attribution)

## Introduction
__Speculation:__ Any piece of information superscripted with a question mark (?) is speculation and should not be treated as fact.

## Data Types
This section describes the various data types that will be used throughout this document.

Types may be appended with `[n]` in order to denote a contiguous array of said type where `n` is the number of elements. For example, `U8[4]` denotes an array of four unsigned 8-bit integers.

### Basic Data Types
These are the "atomic" types as all other structures are built using these fundamental types.

| Type | Description |
| --- | --- |
| U8 | An unsigned 8-bit integer. |
| VINT | An integer of variable size. See [Variable-Size Integers](#variable-size-integers) for more information. |

#### Variable-Size Integers
LCF files predominantly use integers of variable size instead of the more common fixed-length formats, such as 32-bit integers. These types of integers can be thought of as a form of compression: integers will only take up the necessary amount of bytes needed in order to encode a particular value. This is in contrast to fixed-size formats as they will always be represented using the same number of bits regardless of the number being represented.

For example, the value `25` will be represented using a single byte; `123456` will take up two bytes.

Below is a piece of pseudocode that reads and decodes these variable-length integers into a fixed format:
```python
u32 read_variable_integer(reader):
    u32 ret = 0
    
    while true:
        u8 b = reader.read_u8()
        
        ret <<= 7
        ret |= b & 0x7F
        
        if (b & 0x80) == 0:
            break
    
    return ret
```
(This code was adapted from the [gabien-app-r48](https://github.com/20kdc/gabien-app-r48) repository.)

### Complex Data Types
These types are built using a combination of [Basic Data Types](#basic-data-types).

#### String Values
`STRING` values represent a length-prepended string of characters. These are used to represent all forms of textual data.

| Field | Type | Description |
| --- | --- | --- |
| Length | VINT | The length of the string measured in bytes. |
| Characters | U8[`Length`] | The array of characters that make up the string. |

No guarantee is made about the encoding of the characters<sup>1</sup>; therefore, it is up to the implementor's discretion as to which one to use.

<sup>1</sup> Most games using the Japanese language are encoded in [Shift JIS](https://en.wikipedia.org/wiki/Shift_JIS).

## LMT File Structure
This section gives an overview of LMT files as a whole. All LMT files will follow the same basic format:

| Field | Type | Description |
| --- | --- | --- |
| Signature | STRING | This field is used to determine if a file claims to be a valid LMT file. The value of this field should always be "LcfMapTree". |
| MapInfoCount | VINT | The number of [Map Info Structures](#map-info-structure) present. |
| MapInfos | [Map Info](#map-info-structure) [`MapInfoCount`] | An array of map information. |
| MapOrderCount | VINT | The number of map orderings present. |
| MapOrders | VINT[`MapOrderCount`] | This array holds the orderings for the maps presented in `MapInfos`. Each element corresponds to a map ID, and the orderings are stored from first map to last map. |
| Active Node | VINT | |
| MapStart | [Map Start](#map-start-structure) | This field holds map starting information, such as starting position. |

## Map Info Structure
Map info is represented using a tag-based model where each property is represented using a tag. No guarantee is made about the presense of each tag. In the case of a missing tag, the property it represents is expected to take on a specified default value. The order in which the tags appear are guaranteed to be in the listed order<sup>?</sup>. See [Tags](#tags) for more information about tags.

| Field | Type | Default Value | Description |
| --- | --- | --- | --- |
| MapID | VINT | Always present. | The map's unique ID. `0` is usually the root map and shouldn't be treated as a playable map; it is used to provide top-level information<sup>2</sup>. |
| MapName | [Map Name Tag](#map-name-tag) | An empty string. Should always be present<sup>?</sup>. | The map's name. |
| ParentID | [Parent ID Tag](#parent-id-tag) | 0 | The ID of a parent map. Maps should be thought of as being in a tree hierarchy. |
| Indentation | [Indentation Tag](#indentation-tag) | 0 |  |
| MapType | [Map Type Tag](#map-type-tag) | root (0) | The type of map being described. |
| EditPosX | [Edit Position X Tag](#edit-position-x-tag) | 0 | For editor use only. The camera's x-position within an editor<sup>?</sup>. |
| EditPosY | [Edit Position Y Tag](#edit-position-x-tag) | 0 | For editor use only. The camera's y-position within an editor<sup>?</sup>. |
| EditExpanded | [Edit Expanded Tag](#edit-expanded-tag) | false (0) | For editor use only. |
| MusicType | [Music Type Tag](#music-type-tag) | inherit (0) | How music should be played. |
| Music | [Music Tag](#music-tag) | See [Music Tag](#music-tag) | Properties pertaining to music, such as volume and tempo. |
| BackgroundType | [Background Type Tag](#background-type-tag) | inherit (0) | The type of background to display. |
| BackgroundName | [Background Name Tag](#background-name-tag) | An empty string. Should always be present<sup>?</sup>. | The filename of the background to display. |
| TeleportFlag | [Teleport Flag Tag](#teleport-flag-tag) | inherit (0) |  |
| EscapeFlag | [Escape Flag Tag](#escape-flag-tag) | inherit (0) |  |
| SaveFlag | [Save Flag Tag](#save-flag-tag) | inherit (0) |  |
| Encounters | [Encounters Tag](#encounters-tag) | An empty array. Should always be present<sup>?</sup>. | An array of encounters within the map. |
| EncounterSteps | [Encounter Steps Tag](#encounter-steps-tag) | 25 | The steps for encounters. |
| AreaRectangle | [Area Rectangle Tag](#area-rectangle-tag) | [0, 0, 0, 0]. Should always be present<sup>?</sup>. | The map area rectangle. A regular map has a rectangle of [0, 0, 0, 0]. |
| End | [End Tag](#end-tag) | Always present. | Indicates the end of a map info structure. |

<sup>2</sup> Historically, the name of the root map was used to determine a game's title. This, however, remains mostly an ancient artifact as game titles are now determined by an accompanying INI file (`RPG_RT.ini`).

## Map Start Structure

## Tags

## Attribution
In addition to personal digging, information within this document is also based on code within the [gabien-app-r48](https://github.com/20kdc/gabien-app-r48) repository.

RPG Maker is property of Enterbrain, Inc. and Kadokawa Corporation.
