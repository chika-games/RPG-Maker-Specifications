# LCF Map Tree Specification (LMT)
## Table of Contents
* Table of Contents
* [Introduction](#introduction)
* [Data Types](#data-types)
    * [Basic Data Types](#basic-data-types)
        * [Variable-Size Integers](#variable-size-integers)
    * [Complex Data Types](#complex-data-types)
        * [String Type](#string-type)
* [LMT File Structure](#lmt-file-structure)
* [Map Info Structure](#map-info-structure)
* [Map Start Structure](#map-start-structure)
* [Example Map Hierarchy](#example-map-hierarchy)
* [Tags](#tags)
   * [Map Info Tags](#map-info-tags)
   * [Map Start Tags](#map-start-tags)
   * [Music Tags](#music-tags)
   * [Encounter Tags](#encounter-tags)
* [Legal Information](#legal-information)

## Introduction
LCF Map Tree files are used to store map properties, game start information, and map orderings for RPG Maker 2000/2003 games.

This specification is part of a larger collection that can be found at the following URL: [https://github.com/chika-games/RPG-Maker-Specifications](https://github.com/chika-games/RPG-Maker-Specifications)

## Data Types
This section describes the various data types that will be used throughout this specification.

Little-endian byte order is assumed for all types; the least significant byte is stored first, and the most significant byte is stored last.

Types may be appended with `[n]` in order to denote a contiguous array of the type with `n` being the number of elements. For example, `U8[4]` denotes an array of four unsigned 8-bit integers.

### Basic Data Types
These types are considered basic as all other data types are constructed using these.

| Type | Description |
| --- | --- |
| U8 | An unsigned 8-bit integer. |
| U32 | An unsigned 32-bit integer. |
| VINT | An unsigned integer of variable size. See [Variable-Size Integers](#variable-size-integers). |

#### Variable-Size Integers
LCF files predominantly use 7-bit encoded integers instead of the more common fixed-length formats. These types of integers will only take up the necessary amount of bytes needed in order to encode its value. For example, the value `25` will be represented using a single byte; `123456` will take up two bytes. This helps reduce the overall size of the file.

More specifically, bytes will contribute up to 7 bits worth of useful information, and the eighth bit will indicate whether or not another byte is to follow.

Below is some pseudocode that reads and decodes these variable-length integers into a fixed-length (`U32`) format:
```rust
u32 read_variable_integer(reader) {
    u32 ret = 0;
    
    while true {
        u8 b = reader.read_u8();
        
        ret <<= 7;
        ret |= b & 0x7F;
        
        if (b & 0x80) == 0 {
            break;
        }
    }
    
    return ret;
}
```

### Complex Data Types
These types are built using a combination of [Basic Data Types](#basic-data-types).

#### String Type
The `STRING` type represent a length-prepended string of characters. These are used for all forms of textual data.

| Field | Type | Description |
| --- | --- | --- |
| Length | VINT | The length of the string measured in bytes. |
| Characters | U8[`Length`] | The array of characters that make up the string. |

__Note:__ RPG Maker has no standard encoding for strings. It is up to the implementor's discretion as to which encoding to use. (Newer games will usually use UTF-8; Japanese games will tend to use [Shift JIS](https://en.wikipedia.org/wiki/Shift_JIS).)

## LMT File Structure
This section details the overall structure of an LMT file.

LMT files can be viewed as having a tag-based structure similar to SWF files. This allows easier traversal of LMT files as unwanted tags can easily be skipped using their size field. See [Tags](tags) for a list of all relevant tags.

| Field | Type | Description |
| --- | --- | --- |
| Signature | STRING | This field is the file's signature. The value of this field should always be "LcfMapTree". |
| MapInfoCount | VINT | The number of [Map Info Structures](#map-info-structure). |
| MapInfos | [Map Info](#map-info-structure) [`MapInfoCount`] | An array of information for all of a game's maps. |
| MapOrderCount | VINT | The number of map orderings. |
| MapOrders | VINT[`MapOrderCount`] | This array holds the hierarchical orderings for all of a game's maps. Each element corresponds to a map ID, and the orderings are stored from first map to last map. |
| ActiveNode | VINT | For editor use only. The value of this field is the ID of the last active map. Editors may use this to re-open the last active map when opening a project. |
| MapStart | [Map Start](#map-start-structure) | This field holds game start information, such as starting positions. |

## Map Info Structure
This section details the Map Info Structure in its entirety. In practice, not all of the listed tags will be present, though the order should be the same. If a tag is missing, then the property it represents should to take on the specified default value.

| Field | Type | Default Value | Description |
| --- | --- | --- | --- |
| MapID | VINT | Always present. | The map's unique ID. `0` is usually the root map and shouldn't be treated as an ordinary playable map<sup>1</sup>. |
| MapName | [Map Name Tag](#map-name-tag) | Always present. | The map's name. |
| ParentID | [Parent ID Tag](#parent-id-tag) | 0 | The ID of a parent map; `0` means this is a top-level map (parent is root). |
| Indentation | [Indentation Tag](#indentation-tag) | 0 (root); 1 (non-root) | The map's hierarchical indentation; indicates the number of parent maps. `0` is reserved for root maps, `1` is for top-level maps (direct children of the root map), etc. See [Example Map Hierarchy](#example-map-hierarchy). |
| MapType | [Map Type Tag](#map-type-tag) | Map (1) | The type of map being described. |
| EditPosX | [Edit Position X Tag](#edit-position-x-tag) | 0 | For editor use only. This represents the editor's last x-position when this map was last edited. |
| EditPosY | [Edit Position Y Tag](#edit-position-x-tag) | 0 | For editor use only. This represents the editor's last y-position when this map was last edited. |
| EditExpanded | [Edit Expanded Tag](#edit-expanded-tag) | False (0) | For editor use only. |
| MusicType | [Music Type Tag](#music-type-tag) | See tag's section. | How music should be played. |
| Music | [Music Tag](#music-tag) | See [Music Tag](#music-tag) | The music to play and its properties. |
| BackgroundType | [Background Type Tag](#background-type-tag) | See tag's section. | The type of background to display. |
| BackgroundName | [Background Name Tag](#background-name-tag) | "backdrop" or an empty string. | The filename of the background to display. `backdrop` is usually the default value editor's use. |
| TeleportFlag | [Teleport Flag Tag](#teleport-flag-tag) | See [Teleport Flag Tag](#teleport-flag-tag) | Determines whether or not teleporting out of the map is allowed. |
| EscapeFlag | [Escape Flag Tag](#escape-flag-tag) | See [Escape Flag Tag](#escape-flag-tag) | Determines whether or not escaping out of the map is allowed. |
| SaveFlag | [Save Flag Tag](#save-flag-tag) | See [Save Flag Tag](#save-flag-tag) | Determines whether or not saving is always allowed within the map. |
| Encounters | [Encounters Tag](#encounters-tag) | An empty array. | An array of random enemy encounters within the map. |
| EncounterSteps | [Encounter Steps Tag](#encounter-steps-tag) | 25 | The steps between each random encounter. |
| AreaRectangle | [Area Rectangle Tag](#area-rectangle-tag) | [0, 0, 0, 0] | The size of a map area measured in pixels. |
| End | [End Tag](#end-tag) | Always present. | Indicates the end of the map info structure. |

<sup>1</sup> The root map forms the top-most part of the map hierarchy; all maps are children to the root. Additionally, the name of the root map was once used to determine a game's title. However, this is remains a historical artifact as game titles are now determined by an accompanying INI file (`RPG_RT.ini`).

## Map Start Structure
This section details the Map Start Structure in its entirety. In practice, not all of the listed tags will be present, though the order should be the same. If a tag is missing, then the property it represents should to take on the specified default value.

| Field | Type | Default Value | Description |
| --- | --- | --- | --- |
| PartyMapID | [Party Map ID Tag](#party-map-id-tag) | Required to play. | The ID of the player's starting map. |
| PartyX | [Party X Tag](#party-x-tag) | Required to play. | The player's starting x-position within the starting map. |
| PartyY | [Party Y Tag](#party-y-tag) | Required to play. | The player's starting y-position within the starting map. |
| SkiffMapID | [Skiff Map ID Tag](#skiff-map-id-tag) | 0 | The ID of the map to spawn the skiff in. |
| SkiffX | [Skiff X Tag](#skiff-x-tag) | 0 | The skiff's starting x-position within its starting map. |
| SkiffY | [Skiff Y Tag](#skiff-y-tag) | 0 | The skiff's starting y-position within its starting map. |
| ShipMapID | [Ship Map ID Tag](#ship-map-id-tag) | 0 | The ID of the map to spawn the ship in. |
| ShipX | [Ship X Tag](#ship-x-tag) | 0 | The ship's starting x-position within its starting map. |
| ShipY | [Ship Y Tag](#ship-y-tag) | 0 | The ship's starting y-position within its starting map. |
| AirshipMapID | [Airship Map ID Tag](#airship-map-id-tag) | 0 | The ID of the map to spawn the airship in. |
| AirshipX | [Airship X Tag](#airship-x-tag) | 0 | The airship's starting x-position within its starting map. |
| AirshipY | [Airship Y Tag](#airship-y-tag) | 0 | The airship's starting y-position within its starting map. |

## Example Map Hierarchy
This is an example hierarchy to help illustrate various fields of an LMT file. Notice all maps are children to the root map.

```
. My Game (root; indentation=0)
+-- Map 1 (parent=0; indentation=1)
|   +-- Map 2 (parent=1; indentation=2)
|   |   +-- Map 5 (parent=2; indentation=3)
|   |   +-- Map 4 (parent=1; indentation=2)
+-- Map 3 (parent=0; indentation=1)
```

The orderings for these maps should then be [__0__, __1__, __2__, __5__, __4__, __3__].

## Tags
All tags have an ID followed by a size except for [End Tags](#end-tag) and [Monster Group Tags](#monster-group-tag).

Basic format:

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | The tag's ID. |
| TagSize | VINT | The size of the tag's data measured in bytes. |

#### End Tag
Marks the end of a structure or specific tags. This tag only has an ID.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 0. |

---

### Map Info Tags
These tags are used in the [Map Info Structure](#map-info-structure).

#### Map Name Tag
This tag stores the name of a map.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 1. |
| MapName | STRING | The map's name. |

(`STRING` is used here for compactness; it consists of a `VINT` followed by a character array, so the general tag format still applies here.)

#### Parent ID Tag
This tag stores the ID of a parent map.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 2. |
| TagSize | VINT | The size of `ParentID` measured in bytes. |
| ParentID | VINT | The parent map's ID. `0` means there is no parent map. |

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
| MapType | VINT | The map's type. Map types are listed below. |

##### Map Types
| Type | Value | Description |
| --- | --- | --- |
| Root | 0 | A root map. |
| Map | 1 | A regular map. |
| Area | 2 | An area of a map. |

#### Edit Position X Tag
This tag is for editor use only. This represents the editor's last x-position when a particular map was last edited.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 5. |
| TagSize | VINT | The size of `EditPosX` measured in bytes. |
| EditPosX | VINT | The editor's last x-position. |

#### Edit Position Y Tag
This tag is for editor use only. This represents the editor's last y-position when a particular map was last edited.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 6. |
| TagSize | VINT | The size of `EditPosY` measured in bytes. |
| EditPosY | VINT | The editor's last y-position. |

#### Edit Expanded Tag
This tag is for editor use only.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 7. |
| TagSize | VINT | The size of `EditExpanded` measured in bytes. |
| EditExpanded | VINT | This field may be treated like a boolean value: false when zero and true when nonzero. |

#### Music Type Tag
This tag specifies how music should be played within a map.

The default music type should be Event (1) for top-level maps (maps whose parent is the root map) and Inherit (0) for all other maps.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 11. |
| TagSize | VINT | The size of `MusicType` measured in bytes. |
| MusicType | VINT | The map's music type. Special music types are listed below. |

##### Music Types
| Type | Value | Description |
| --- | --- | --- |
| Inherit | 0 | Inherit the music. |
| Event | 1 | Music provided through an event. |
| Specified | 2 | Music is specified explicitly. |

#### Music Tag
This tag specifies various music properties for a map. If one of the listed tags is missing, then its default value is assumed.

| Field | Type | Default | Description |
| --- | --- | --- | --- |
| TagID | VINT | Always present. | TagID is 12. |
| TagSize | VINT | Always present. | The total size of the below fields measured in bytes. |
| MusicName | [Music Name Tag](#music-name-tag) | "(OFF)" | The filename of the music to be played. |
| MusicFadeTime | [Music Fade Time Tag](#music-fade-time-tag) | 0 | The fade time for the music. 0 means no fade. |
| MusicVolume | [Music Volume Tag](#music-volume-tag) | 100 | The volume of the music. |
| MusicTempo | [Music Tempo Tag](#music-tempo-tag) | 100 | The tempo of the music. |
| MusicBalance | [Music Balance Tag](#music-balance-tag) | 50 | The left-right balance of the music. 50 is central. |
| End | [End Tag](#end-tag) | Always present. | Indicates the end of the music tag. |

#### Background Type Tag
This tag specifies the type of background within a map.

The default value of `BackgroundType` should be Terrain (1) for top-level maps (maps whose parent is the root map) and Inherit (0) for all other maps.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 21. |
| TagSize | VINT | The size of `BackgroundType` measured in bytes. |
| BackgroundType | VINT | The map's background type. Special background types are listed below. |

##### Background Types
| Type | Value | Description |
| --- | --- | --- |
| Inherit | 0 | Inherit the background. |
| Terrain | 1 | Use the terrain settings. |
| Specified | 2 | Background is specified explicitly. |

#### Background Name Tag
This tag specifies the filename of a map's background.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 22. |
| BackgroundName | STRING | The background's filename. |

(`STRING` is used here for compactness; it consists of a `VINT` followed by a character array, so the general tag format still applies here.)

#### Teleport Flag Tag
Determines whether or not teleporting out of the map is allowed. For top-level maps (maps whose parent is the root map), the default value of `TeleportFlag` is 1; otherwise, for maps with a non-root parent, the default value is 0.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 31. |
| TagSize | VINT | The size of `TeleportFlag` measured in bytes. |
| TeleportFlag | VINT | The map's teleport flag. Special values are listed below. |

##### Teleport Flag Values
| Type | Value | Description |
| --- | --- | --- |
| Inherit | 0 | Use the same value as the map's parent. |
| Allow | 1 | Allow teleporting. |
| Forbid | 2 | Forbid teleporting. |

#### Escape Flag Tag
Determines whether or not escaping out of the map is allowed. For top-level maps (maps whose parent is the root map), the default value of `EscapeFlag` is 1; otherwise, for maps with a non-root parent, the default value is 0.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 32. |
| TagSize | VINT | The size of `EscapeFlag` measured in bytes. |
| EscapeFlag | VINT | The map's escape flag. Special values are listed below. |

##### Escape Flag Values
| Type | Value | Description |
| --- | --- | --- |
| Inherit | 0 | Use the same value as the map's parent. |
| Allow | 1 | Allow escaping. |
| Forbid | 2 | Forbid escaping. |

#### Save Flag Tag
Determines whether or not saving is allowed within the map. For top-level maps (maps whose parent is the root map), the default value of `EscapeFlag` is 1; otherwise, for maps with a non-root parent, the default value is 0.

If `SaveFlag` is set to `Allow`, then a save option will be available in the pause menu; otherwise, saving can only occur through events.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 33. |
| TagSize | VINT | The size of `SaveFlag` measured in bytes. |
| SaveFlag | VINT | The map's save flag. Special values are listed below. |

##### Save Flag Values
| Type | Value | Description |
| --- | --- | --- |
| Inherit | 0 | Use the same value as the map's parent. |
| Allow | 1 | Allow saving. |
| Forbid | 2 | Forbid saving. |

#### Encounters Tag
This tag describes the type of enemy encounters within a map.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 41. |
| TagSize | VINT | The total size of the following fields measured in bytes. |
| EncounterCount | VINT | The number of encounters. |
| Encounters | [Monster Group Tag](#monster-group-tag) [`EncounterCount`] | The encounters. |

#### Encounter Steps Tag
This tag specifies the encounter steps for a map.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 44. |
| TagSize | VINT | The size of `EncounterSteps` measured in bytes. |
| EncounterSteps | VINT | The encounter steps for a map. |

#### Area Rectangle Tag
This tag specifies the boundaries of an area of a map. This only applies to maps with a `MapType` of `Area`; other kinds of maps have a rectangle of [0, 0, 0, 0].

The coordinates are measured in pixels and begin in the top-left corner of a map. For example, an area rectangle of [0, 0, 100, 100] would completely cover a 100x100 map.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 51. |
| TagSize | VINT | Should be 16. |
| Left | U32 | The left-coordinate of the rectangle. |
| Top | U32 | The top-coordinate of the rectangle. |
| Right | U32 | The right-coordinate of the rectangle. |
| Bottom | U32 | The bottom-coordinate of the rectangle. |

---

### Map Start Tags
These tags are used in the [Map Start Structure](#map-start-structure).

#### Party Map ID Tag
This tag specifies the player's starting map ID.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 1. |
| TagSize | VINT | The size of `MapID` measured in bytes. |
| MapID | VINT | The starting map ID. |

#### Party X Tag
This tag specifies the player's starting x-position within the starting map.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 2. |
| TagSize | VINT | The size of `XPos` measured in bytes. |
| XPos | VINT | The starting x-position. |

#### Party Y Tag
This tag specifies the player's starting y-position within the starting map.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 3. |
| TagSize | VINT | The size of `YPos` measured in bytes. |
| YPos | VINT | The starting y-position. |

#### Skiff Map ID Tag
This tag specifies the skiff's starting map ID.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 11. |
| TagSize | VINT | The size of `MapID` measured in bytes. |
| MapID | VINT | The starting map ID. |

#### Skiff X Tag
This tag specifies the skiff's starting x-position within its starting map.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 12. |
| TagSize | VINT | The size of `XPos` measured in bytes. |
| XPos | VINT | The starting x-position. |

#### Skiff Y Tag
This tag specifies the skiff's starting y-position within its starting map.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 13. |
| TagSize | VINT | The size of `YPos` measured in bytes. |
| YPos | VINT | The starting y-position. |

#### Ship Map ID Tag
This tag specifies the ship's starting map ID.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 21. |
| TagSize | VINT | The size of `MapID` measured in bytes. |
| MapID | VINT | The starting map ID. |

#### Ship X Tag
This tag specifies the ship's starting x-position within its starting map.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 22. |
| TagSize | VINT | The size of `XPos` measured in bytes. |
| XPos | VINT | The starting x-position. |

#### Ship Y Tag
This tag specifies the ship's starting y-position within its starting map.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 23. |
| TagSize | VINT | The size of `YPos` measured in bytes. |
| YPos | VINT | The starting y-position. |

#### Airship Map ID Tag
This tag specifies the airship's starting map ID.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 31. |
| TagSize | VINT | The size of `MapID` measured in bytes. |
| MapID | VINT | The starting map ID. |

#### Airship X Tag
This tag specifies the airship's starting x-position within its starting map.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 32. |
| TagSize | VINT | The size of `XPos` measured in bytes. |
| XPos | VINT | The starting x-position. |

#### Airship Y Tag
This tag specifies the airship's starting y-position within its starting map.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 33. |
| TagSize | VINT | The size of `YPos` measured in bytes. |
| YPos | VINT | The starting y-position. |

---

### Music Tags
These tags are used within the [Music Tag](#music-tag).

#### Music Name Tag
This tag specifies the filename of the music to be played in a map. If `MusicName` is "(OFF)", then no music will automatically play within the map.

| Field | Type | Description |
| --- | --- | --- |
| TagID | VINT | TagID is 1. |
| MusicName | STRING | The music's filename. |

(`STRING` is used here for compactness; it consists of a `VINT` followed by a character array, so the general tag format still applies here.)

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

---

### Encounter Tags
These tags are used within the [Encounters Tag](#encounters-tag).

#### Monster Group Tag
| Field | Type | Description |
| --- | --- | --- |
| EncounterIndex | VINT | The position of this encounter within the encounter array starting at `1`. |
| N/A | VINT | Should be `1`. |
| GroupBytes | VINT | The size of `GroupID` measured in bytes. |
| GroupID | VINT | The type of monster group to use. |
| End | [End Tag](#end-tag) | Indicates the end of the monster group tag. |

## Legal Information
This document is provided under the [CC BY-SA 4.0](http://creativecommons.org/licenses/by-sa/4.0/) license.

The RPG Maker trademark and copyright are property of Enterbrain, Inc. and Kadokawa Corporation.

All rights belong to their respective owners.
