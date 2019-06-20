# LCF Map Tree Specification (LMT)
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
   * [Map Info Tags](#map-info-tags)
   * [Music Tags](#music-tags)
   * [Encounter Tags](#encounter-tags)
* [Document Changes](#document-changes)
* [Attribution](#attribution)

## Introduction
__Speculation:__ Any piece of information superscripted with a question mark (?) is speculation and should not be treated as fact.

This specification is part of a larger collection that can be found at the following link: [https://github.com/napen123/rpgmaker-asset-specs](https://github.com/napen123/rpgmaker-asset-specs)

## Data Types
This section describes the various data types that will be used throughout this document.

Types may be appended with `[n]` in order to denote a contiguous array of said type where `n` is the number of elements. For example, `U8[4]` denotes an array of four unsigned 8-bit integers.

### Basic Data Types
These are the "atomic" types as all other structures are built using these fundamental types.

| Type | Description |
| --- | --- |
| U8 | An unsigned 8-bit integer. |
| U32 | An unsigned 32-bit integer. |
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
| End | [End Tag](#end-tag) | Always present. | Indicates the end of the map info structure. |

<sup>2</sup> Historically, the name of the root map was used to determine a game's title. This, however, remains mostly an ancient artifact as game titles are now determined by an accompanying INI file (`RPG_RT.ini`).

## Map Start Structure

## Tags
All tags have a tag ID followed by a size except for [End Tags](#end-tag) and [Troop Tags](#troop-tag):

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | The tag's ID. |
| TagSize | VINT | The size of the tag's data measured in bytes. |

#### End Tag
Marks the end of some structure or other specific tags. This tag only has an ID.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 0. |

### Map Info Tags
These tags are used in the [Map Info Structure](#map-info-structure).

#### Map Name Tag
This tag stores the name of a map.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 1. |
| TagSize | VINT | The size of the map's name in bytes. |
| MapName | U8[`TagSize`] | The map's name. |

#### Parent ID Tag
This tag stores the ID of a parent map.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 2. |
| TagSize | VINT | The size of `ParentID` measured in bytes. |
| ParentID | VINT | The parent map's ID. 0 means there is no parent map. |

#### Indentation Tag
| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 3. |
| TagSize | VINT | The size of `Indentation` measured in bytes. |
| Indentation | VINT | The map's indentation. |

#### Map Type Tag
This tag specifies the type of map being described.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 4. |
| TagSize | VINT | The size of `MapType` measured in bytes. |
| MapType | VINT | The map's type. Special map types are listed below. |

| Type | Value | Description |
| --- | --- | --- |
| Root | 0 | A root map. |
| Map | 1 | A regular map. |
| Area | 2 | An area map. |

#### Edit Position X Tag
This tag is for editor use only. The camera's x-position within an editor<sup>?</sup>.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 5. |
| TagSize | VINT | The size of `EditPosX` measured in bytes. |
| EditPosX | VINT | The editor x-position. |

#### Edit Position Y Tag
This tag is for editor use only. The camera's y-position within an editor<sup>?</sup>.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 6. |
| TagSize | VINT | The size of `EditPosY` measured in bytes. |
| EditPosY | VINT | The editor y-position. |

#### Edit Expanded Tag
This tag is for editor use only.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 7. |
| TagSize | VINT | The size of `EditExpanded` measured in bytes. |
| EditExpanded | VINT | This field may be treated like a boolean value: false when zero and true when nonzero. |

#### Music Type Tag
This tag specifies how music should be played within a map.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 11. |
| TagSize | VINT | The size of `MusicType` measured in bytes. |
| MusicType | VINT | The map's music type. Special music types are listed below. |

| Type | Value | Description |
| --- | --- | --- |
| Inherit | 0 | Inherit the music. |
| Event | 1 | Music specified through an event. |
| Specified | 2 | Music is specified explicitly. |

#### Music Tag
This tag specifies various music properties for a map. If one of the listed tags is missing, then its default value is assumed.

| Field | Type | Default | Description |
| --- | --- | --- | --- |
| TagID | VINT | Always present. | TagID is 12. |
| TagSize | VINT | Always present. | The total size of the below tags measured in bytes. |
| MusicName | [Music Name Tag](#music-name-tag) | An empty string. | The filename of the music to be played. |
| MusicFadeTime | [Music Fade Time Tag](#music-fade-time-tag) | 0 | The fade time for the music. 0 means there is no fading. |
| MusicVolume | [Music Volume Tag](#music-volume-tag) | 100 | The volume of the music. |
| MusicTempo | [Music Tempo Tag](#music-tempo-tag) | 100 | The tempo of the music. |
| MusicBalance | [Music Balance Tag](#music-balance-tag) | 50 | The left-right balance of the music. 50 means centered. |
| End | [End Tag](#end-tag) | Always present. | Indicates the end of the music tag. |

#### Background Type Tag
This tag specifies the type of background within a map.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 21. |
| TagSize | VINT | The size of `BackgroundType` measured in bytes. |
| BackgroundType | VINT | The map's background type. Special background types are listed below. |

| Type | Value | Description |
| --- | --- | --- |
| Inherit | 0 | Inherit the background. |
| TerrainLdb | 1 |  |
| Specified | 2 | Background is specified explicitly. |

#### Background Name Tag
This tag specifies the filename of a map's background.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 22. |
| TagSize | VINT | The size of the background's name in bytes. |
| BackgroundName | U8[`TagSize`] | The background's filename. |


#### Teleport Flag Tag
| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 31. |
| TagSize | VINT | The size of `TeleportFlag` measured in bytes. |
| TeleportFlag | VINT | The map's teleport flag. Special values are listed below. |

| Type | Value | Description |
| --- | --- | --- |
| Inherit | 0 | Inherit the flag's value. |
| True | 1 | The flag is set. |
| False | 2 | The flag is unset. |

#### Escape Flag Tag
| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 32. |
| TagSize | VINT | The size of `EscapeFlag` measured in bytes. |
| EscapeFlag | VINT | The map's escape flag. Special values are listed below. |

| Type | Value | Description |
| --- | --- | --- |
| Inherit | 0 | Inherit the flag's value. |
| True | 1 | The flag is set. |
| False | 2 | The flag is unset. |

#### Save Flag Tag
| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 33. |
| TagSize | VINT | The size of `SaveFlag` measured in bytes. |
| SaveFlag | VINT | The map's save flag. Special values are listed below. |

| Type | Value | Description |
| --- | --- | --- |
| Inherit | 0 | Inherit the flag's value. |
| True | 1 | The flag is set. |
| False | 2 | The flag is unset. |

#### Encounters Tag
This tag outlines any encounters within a map.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 41. |
| TagSize | VINT | The total size of the following fields measured in bytes. |
| EncounterCount | VINT | The number of encounters. |
| Encounters | [Troop Tag](#troop-tag) [`EncounterCount`] | The troops involved in each encounter. |

#### Encounter Steps Tag
This tag specifies the encounter steps for a map.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 44. |
| TagSize | VINT | The size of `EncounterSteps` measured in bytes. |
| EncounterSteps | VINT | The encounter steps for a map. 0 means encounters are disabled. |

#### Area Rectangle Tag
This tag specifies the area rectangle for a map. Regular maps have a rectangle of [0, 0, 0, 0].

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 51. |
| TagSize | VINT | Should always be 16<sup>?</sup>. |
| Left | U32 | The left-coordinate of the rectangle. |
| Top | U32 | The top-coordinate of the rectangle. |
| Right | U32 | The right-coordinate of the rectangle. |
| Bottom | U32 | The bottom-coordinate of the rectangle. |

### Music Tags
These tags are used within the [Music Tag](#music-tag).

#### Music Name Tag
This tag specifies the filename of the music to be played in a map.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 1. |
| TagSize | VINT | The size of the music's filename in bytes. |
| MapName | U8[`TagSize`] | The music's filename. |

#### Music Fade Time Tag
This tag specifies the fade time for a map's music.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 2. |
| TagSize | VINT | The size of `FadeTime` measured in bytes. |
| FadeTime | VINT | The music's fade time. |

#### Music Volume Tag
This tag specifies the volume for a map's music.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 3. |
| TagSize | VINT | The size of `Volume` measured in bytes. |
| Volume | VINT | The music's volume. |

#### Music Tempo Tag
This tag specifies the tempo for a map's music.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 4. |
| TagSize | VINT | The size of `Tempo` measured in bytes. |
| Tempo | VINT | The music's tempo. |

#### Music Balance Tag
This tag specifies the left-right balance for a map's music.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 5. |
| TagSize | VINT | The size of `Balance` measured in bytes. |
| Balance | VINT | The music's left-right balance. |

### Encounter Tags
These tags are used within the [Encounters Tag](#encounters-tag).

#### Troop Tag<sup>?</sup>
| Field | Type | Description |
| --- | --- | --- |
| TroopIndex | VINT | The troop's index within an encounter array. |
| ??? | VINT |   |
| ??? | VINT |   |
| TroopID | VINT | The troop's unique ID. This can be treated as the type of troop<sup>3</sup>. |
| End | [End Tag](#end-tag) | Always present. | Indicates the end of the troop tag. |

<sup>3</sup> For example, a game may understand 15 to be green ooze.

## Document Changes

## Attribution
20kdc's [gabien-app-r48](https://github.com/20kdc/gabien-app-r48).

RPG Maker is property of Enterbrain, Inc. and Kadokawa Corporation.
