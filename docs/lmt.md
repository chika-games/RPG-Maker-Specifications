---
layout: default
---

# LCF Map Tree File Specification
## Introduction
LCF Map Tree (LMT) files are used to store map properties, game start information, and map orderings for RPG Maker 2000/2003 (RM2k/3) games.
These LMT files use a tag-based format similar to the one used in e.g. Adobe SWF files.

RM2k/3 games typically only have one LMT file: `RPG_RT.lmt`. This is located within the same directory as the game's executable file.

## Data Types
LMT files use little-endian byte ordering; the least significant byte is stored first, and the most significant byte is stored last.

Types may be appended with `[n]` to denote a contiguous array of the type in question, where `n` indicates the number of elements.
For example, `U8[4]` denotes a contiguous array of four unsigned 8-bit integers.

These data types are used within the actual structural types that determine the overall the structure of LMT files.

### Basic Data Types
The following table lists all of the basic data types that make up all other data types.

| Type   | Description                 |
|:-------|:----------------------------|
| `U8`   | An unsigned 8-bit integer.  |
| `U32`  | An unsigned 32-bit integer. |
| `EINT` | A 7-bit encoded integer.    |

#### Encoded Integers
LCF files make extensive use of 7-bit encoded integers instead of the more common fixed-length formats.
These will typically only use the minimum number of bytes needed to encode a particular value.
For example, _123456_ will be encoded as three bytes. This helps reduce the overall size of LCF files.

As the name implies, only the lower seven bits of an encoded integer will contribute to its overall value;
the eighth (high) bit is used to determine whether or not the there is another byte that should be read and decoded.

Below is some pseudocode for reading and decoding these encoded integers into a fixed-length format (`U32`):

```c
u32 read_variable_integer(reader) {
    u32 ret = 0;

    while true {
        u8 byte = reader.read_u8();

        ret <<= 7;
        ret |= byte & 0x7F;

        if (byte & 0x80) == 0 {
            break;
        }
    }

    return ret;
}
```

### Compound Data Types
Compound data types consist of a combination of basic data types.
These correspond to structures and/or records in most programming languages.

#### STRING Type
The `STRING` type represents a length-prepended string of characters. These are used for all textual data.

**Note:** Strings have no standard encoding. It is up to the runtime's discretion as to which encoding to use;
this is typically based on the operating system's current locale. Japanese games will typically use [Shift JIS](https://en.wikipedia.org/wiki/Shift_JIS).

| Field  | Type         | Description                             |
|:-------|:-------------|:----------------------------------------|
| Length | `EINT`       | The length of the string in bytes.      |
| Chars  | `U8[Length]` | The characters that make up the string. |

## LCF Map Tree File Structure
LMT files are stored in binary format.

The following table describes the overall structure of an LMT file.

| Field         | Type                                            | Description                                               |
|:--------------|:------------------------------------------------|:----------------------------------------------------------|
| Signature     | `STRING`                                        | The file's signature; this should always be "LcfMapTree". |
| MapInfoCount  | `EINT`                                          | The number of map info structures present.                |
| MapInfos      | [Map Info](#map-info-structure)`[MapInfoCount]` | An array of map info structures.                          |
| MapOrderCount | `EINT`                                          | The number of map orderings present.                      |
| MapOrders     | `EINT[MapOrderCount]`                           | The hierarchical orderings of a game's maps.              |
| ActiveNode    | `EINT`                                          | The ID of the last saved/edited map.                      |
| MapStart      | [Map Start](#map-start-structure)               | Game start information, such as starting positions.       |

All maps are arranged in a hierarchy and are children to the first map in the `MapInfos` array called the root map.
This root map is not meant to be playable and is simply meant to be the default parent to all maps in the hierarchy.
In older versions of the RM2k/3 runtimes, the name of the root map was also used to determine the name of the game's window.
This functionality has since been deprecated in favor of a [configuration file](./config.html).

`ActiveNode` is used by the RM2k/3 editors to keep track of the last saved/edited map.
This allows them to automatically open the last edited map when launching.

## Map Info Structure
The following table describes the layout of `Map Info` structures.

**Note:** In practice, not all of this structure's tag-based fields will be present. This is presumably to reduce the overall size of LMT files.
If such a field is missing, then its respective default value should be used.

| Field          | Type                                        | Default Value                                               | Description                                                                                                                    |
|:---------------|:--------------------------------------------|:------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------|
| MapID          | `EINT`                                      | Always present.                                             | The map's unique ID. `0` is always the root map's ID.                                                                          |
| MapName        | [Map Name Tag](#map-name-tag)               | Always present.                                             | The map's name.                                                                                                                |
| ParentID       | [Parent ID Tag](#parent-id-tag)             | 0                                                           | The ID of a map's parent. `0` means it's a top-level map (root is parent).                                                     |
| Indentation    | [Indentation Tag](#indentation-tag)         | 1 (0 if root)                                               | The map's hierarchical indentation; it indicates how deep it is in the hierarchy. See the [example hierarchy](#map-hierarchy). |
| MapType        | [Map Type Tag](#map-type-tag)               | Map (1)                                                     | The type of map.                                                                                                               |
| EditPosX       | [Edit Position X Tag](#edit-position-x-tag) | 0                                                           | The editor's last x-position when the map was last edited.                                                                     |
| EditPosY       | [Edit Position Y Tag](#edit-position-x-tag) | 0                                                           | The editor's last y-position when the map was last edited.                                                                     |
| EditExpanded   | [Edit Expanded Tag](#edit-expanded-tag)     | False (0)                                                   | Whether or not the map was expanded in the editor's tree view (children were visible).                                         |
| MusicType      | [Music Type Tag](#music-type-tag)           | Event (1) for top-level maps; Inherit (0) for child maps.   | How map's background music should be played.                                                                                   |
| Music          | [Music Tag](#music-tag)                     | See tag's section.                                          | The map's background music and its playback settings.                                                                          |
| BackgroundType | [Background Type Tag](#background-type-tag) | Terrain (1) for top-level maps; Inherit (0) for child maps. | The type of background to display while in combat.                                                                             |
| BackgroundName | [Background Name Tag](#background-name-tag) | "backdrop" or ""                                            | The filename of the combat background.                                                                                         |
| TeleportFlag   | [Teleport Flag Tag](#teleport-flag-tag)     | Allow (1) for top-level maps; Inherit (0) for child maps.   | Determines whether or not teleporting out of the map is allowed.                                                               |
| EscapeFlag     | [Escape Flag Tag](#escape-flag-tag)         | Allow (1) for top-level maps; Inherit (0) for child maps.   | Determines whether or not escaping out of the map is allowed.                                                                  |
| SaveFlag       | [Save Flag Tag](#save-flag-tag)             | Allow (1) for top-level maps; Inherit (0) for child maps.   | Determines whether or not saving is allowed within the map.                                                                    |
| Encounters     | [Encounters Tag](#encounters-tag)           | An empty array.                                             | All possible (random) combat encounters that can appear within the map.                                                        |
| EncounterSteps | [Encounter Steps Tag](#encounter-steps-tag) | 25                                                          | The likelihood of having a random encounter.                                                                                   |
| AreaRectangle  | [Area Rectangle Tag](#area-rectangle-tag)   | [0, 0, 0, 0]                                                | The size of a map area measured in pixels. |
| End            | [End Tag](#end-tag)                         | Always present.                                             | Indicates the end of the structure. |

### Map Hierarchy
All maps of an RM2k/3 game are arranged in a parent-child hierarchy where the root map is at the top of the hierarchy.
Below is an example of such a hierarchy that is intended to help illustrate some of the fields of the `Map Info` structure.

```
. My Game (root; indentation=0)
+-- Map 1 (parent=0; indentation=1)
|   +-- Map 2 (parent=1; indentation=2)
|   |   +-- Map 5 (parent=2; indentation=3)
|   +-- Map 4 (parent=1; indentation=2)
+-- Map 3 (parent=0; indentation=1)
```

Notice that the maps are not required to be in sequential order. These arbitrary orderings are what the `MapOrders` field determines.
Additionally, the ordering of the maps are mainly used by the editors and don't necessarily reflect the actual play order.

## Map Start Structure
The following table describes the layout of the `Map Start` structure.
This structure specifies the maps and positions the player and vehicles should spawn at when a new game is created.

**Note:** In practice, not all of this structure's tag-based fields will be present.
This happens when the player or vehicles were not given a starting position within the game (i.e. they're not used in-game).
If the player (Party) is not assigned a starting position, the RM2k/3 runtime will crash with an error when attempting to play.

All x and y positions are measured in map tiles.

| Field        | Type                                      | Description                                                                                                                    |
|:-------------|:------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------|
| PartyMapID   | [Party Map ID Tag](#party-map-id-tag)     | The ID of the map the player should initially spawn in.          |
| PartyX       | [Party X Tag](#party-x-tag)               | The x-position the player should spawn at in their starting map. |
| PartyY       | [Party Y Tag](#party-y-tag)               | The y-position the player should spawn at in their starting map. |
| SkiffMapID   | [Skiff Map ID Tag](#skiff-map-id-tag)     | The ID of the map the skiff vehicle should initially spawn in.   |
| SkiffX       | [Skiff X Tag](#skiff-x-tag)               | The x-position the skiff should spawn at in its starting map.    |
| SkiffY       | [Skiff Y Tag](#skiff-y-tag)               | The y-position the skiff should spawn at in its starting map.    |
| ShipMapID    | [Ship Map ID Tag](#ship-map-id-tag)       | The ID of the map the ship vehicle should initially spawn in.    |
| ShipX        | [Ship X Tag](#ship-x-tag)                 | The x-position the ship should spawn at in its starting map.     |
| ShipY        | [Ship Y Tag](#ship-y-tag)                 | The y-position the ship should spawn at in its starting map.     |
| AirshipMapID | [Airship Map ID Tag](#airship-map-id-tag) | The ID of the map the airship vehicle should initially spawn in. |
| AirshipX     | [Airship X Tag](#airship-x-tag)           | The x-position the airship should spawn at in its starting map.  |
| AirshipY     | [Airship Y Tag](#airship-y-tag)           | The y-position the airship should spawn at in its starting map.  |
| End          | [End Tag](#end-tag)                       | Indicates the end of the structure.                              |

## Tags
All tags begin with the same basic format below except for the [End Tag](#end-tag) and [Monster Group Tag](#monster-group-tag).

Basic format:

| Field   | Type   | Description                                               |
|:--------|:-------|:----------------------------------------------------------|
| TagID   | `EINT` | The tag's semi-unique ID.                                 |
| TagSize | `EINT` | The size of the tag's remaining fields measured in bytes. |

The `TagID` field identifies tags semi-uniquely; some tags may share an ID, but this is never happens within the same structure/context.

The `TagSize` field measures the total number of bytes the rest of the tag's fields take up.
This field is occasionally used by other fields and can be used to skip over recognized tags by simply reading or skipping over `TagSize` number of bytes.

### End Tag
Marks the end of a structure or tag when there are no more relevant fields present.

This tag only has an ID field.

| Field | Type   | Description            |
|:------|:-------|:-----------------------|
| TagID | `EINT` | This will always be 0. |

### Map Info Tags
These tags are used within the [Map Info Structure](#map-info-structure).

#### Map Name Tag
This tag provides the name of a map.

| Field   | Type          | Description                                 |
|:--------|:--------------|:--------------------------------------------|
| TagID   | `EINT`        | This will always be 1.                      |
| TagSize | `EINT`        | The number of characters in `MapName`.      |
| MapName | `U8[TagSize]` | The characters that make up the map's name. |

`TagSize` + `MapName` essentially acts as a [`STRING`](#string-type) type.

#### Parent ID Tag
This tag provides the ID of a map's parent map.

| Field    | Type   | Description                               |
|:---------|:-------|:------------------------------------------|
| TagID    | `EINT` | This will always be 2.                    |
| TagSize  | `EINT` | The size of `ParentID` measured in bytes. |
| ParentID | `EINT` | The ID of the map's parent.               |

#### Indentation Tag
This tag provides a map's indentation in the (Map Hierarchy)[#map-hierarchy].

| Field       | Type          | Description                                  |
|:------------|:--------------|:---------------------------------------------|
| TagID       | `EINT`        | This will always be 3.                       |
| TagSize     | `EINT`        | The size of `Indentation` measured in bytes. |
| Indentation | `EINT`        | The indentation of the map.                  |

#### Map Type Tag
This tag provides a map's type.

| Field   | Type   | Description                              |
|:--------|:-------|:-----------------------------------------|
| TagID   | `EINT` | This will always be 4.                   |
| TagSize | `EINT` | The size of `MapType` measured in bytes. |
| MapType | `EINT` | The map's type. See table below.         |

**Map Type**
| Type | Value | Description       |
|:-----|:------|:------------------|
| Root | 0     | The root map.     |
| Map  | 1     | A regular map.    |
| Area | 2     | An area of a map. |

#### Edit Position X Tag
This specifies the editor's last x-position when a particular map was previously edited.

| Field    | Type   | Description                               |
|:---------|:-------|:------------------------------------------|
| TagID    | `EINT` | This will always be 5.                    |
| TagSize  | `EINT` | The size of `EditPosX` measured in bytes. |
| EditPosX | `EINT` | The editor's last x-position.             |

#### Edit Position Y Tag
This specifies the RM2k/3 editor's last y-position when a particular map was previously edited.

| Field    | Type   | Description                               |
|:---------|:-------|:------------------------------------------|
| TagID    | `EINT` | This will always be 6.                    |
| TagSize  | `EINT` | The size of `EditPosY` measured in bytes. |
| EditPosY | `EINT` | The editor's last y-position.             |

#### Edit Expanded Tag
This specifies whether or not a map was previously "expanded" within the the RM2k/3 editor;
i.e., it's children maps were visible in the (Map Hierarchy)[#map-hierarchy].

| Field        | Type   | Description                                    |
|:-------------|:-------|:-----------------------------------------------|
| TagID        | `EINT` | This will always be 7.                         |
| TagSize      | `EINT` | The size of `EditPosY` measured in bytes.      |
| EditExpanded | `EINT` | Whether or not the map was expanded in-editor. |

#### Music Type Tag
This tag specifies how music should be played within a map.

| Field     | Type   | Description                                |
|:----------|:-------|:-------------------------------------------|
| TagID     | `EINT` | This will always be 11.                    |
| TagSize   | `EINT` | The size of `MusicType` measured in bytes. |
| MusicType | `EINT` | The map's music type. See table below.     |

**Music Type**
| Type      | Value | Description                           |
|:----------|:------|:--------------------------------------|
| Inherit   | 0     | Inherit music type from map's parent. |
| Event     | 1     | Music provided through events.        |
| Specified | 2     | Play a specific song.                 |

#### Music Tag
This tag specifies a particular song and its playback properties.

Not all fields may be present; in this case, the field's default value should be used.

| Field    | Type                                        | Default Value   | Description                                               |
|:---------|:--------------------------------------------|:----------------|:----------------------------------------------------------|
| TagID    | `EINT`                                      | Always present. | This will always be 12.                                   |
| TagSize  | `EINT`                                      | Always present. | The total size of the remaining fields measured in bytes. |
| Name     | [Music Name Tag](#music-name-tag)           | "(OFF)"         | The filename of the song to play.                         |
| FadeTime | [Music Fade Time Tag](#music-fade-time-tag) | 0               | The fade time for the music; 0 means no fade.             |
| Volume   | [Music Volume Tag](#music-volume-tag)       | 100             | The volume of the music.                                  |
| Tempo    | [Music Tempo Tag](#music-tempo-tag)         | 100             | The tempo of the music.                                   |
| Balance  | [Music Balance Tag](#music-balance-tag)     | 50              | The left-right balance of the music; 50 is centered.      |
| End      | [End Tag](#end-tag)                         | Always present. | Indicates the end of the music tag.                       |

#### Background Type Tag
This tag specifies the type of background a map has.

| Field          | Type   | Description                                      |
|:---------------|:-------|:-------------------------------------------------|
| TagID          | `EINT` | This will always be 21.                          |
| TagSize        | `EINT` | The size of `BackgroundType` measured in bytes.  |
| BackgroundType | `EINT` | The map's background type. See table below.      |

**Background Type**
| Type      | Value | Description                           |
|:----------|:------|:--------------------------------------|
| Inherit   | 0     | Inherit music type from map's parent. |
| Terrain   | 1     | Use the terrain settings.             |
| Specified | 2     | Use a specified background image.     |

#### Background Name Tag
This tag provides the filename of a map's background image.

| Field          | Type          | Description                                        |
|:---------------|:--------------|:---------------------------------------------------|
| TagID          | `EINT`        | This will always be 22.                            |
| TagSize        | `EINT`        | The number of characters in `BackgroundName`.      |
| BackgroundName | `U8[TagSize]` | The characters that make up the background's name. |

`TagSize` + `BackgroundName` essentially acts as a [`STRING`](#string-type) type.

#### Teleport Flag Tag
This tag specifies whether or not teleporting is allowed within a map.

| Field        | Type   | Description                                               |
|:-------------|:-------|:----------------------------------------------------------|
| TagID        | `EINT` | This will always be 31.                                   |
| TagSize      | `EINT` | The size of `TeleportFlag` measured in bytes.             |
| TeleportFlag | `EINT` | Whether or not teleportation is allowed. See table below. |

**Teleport Flag Values**
| Type    | Value | Description                             |
|:--------|:------|:----------------------------------------|
| Inherit | 0     | Inherit flag's value from map's parent. |
| Allow   | 1     | Allow teleportation.                    |
| Forbid  | 2     | Forbid teleportation.                   |

#### Escape Flag Tag
This tag specifies whether or not escaping is allowed within a map.

| Field      | Type   | Description                                          |
|:-----------|:-------|:-----------------------------------------------------|
| TagID      | `EINT` | This will always be 32.                              |
| TagSize    | `EINT` | The size of `EscapeFlag` measured in bytes.          |
| EscapeFlag | `EINT` | Whether or not escaping is allowed. See table below. |

**Escape Flag Values**
| Type    | Value | Description                             |
|:--------|:------|:----------------------------------------|
| Inherit | 0     | Inherit flag's value from map's parent. |
| Allow   | 1     | Allow escaping.                         |
| Forbid  | 2     | Forbid escaping.                        |

#### Save Flag Tag
This tag specifies whether or not saving is allowed within a map.

| Field    | Type   | Description                                        |
|:---------|:-------|:---------------------------------------------------|
| TagID    | `EINT` | This will always be 33.                            |
| TagSize  | `EINT` | The size of `SaveFlag` measured in bytes.          |
| SaveFlag | `EINT` | Whether or not saving is allowed. See table below. |

**Save Flag Values**
| Type    | Value | Description                             |
|:--------|:------|:----------------------------------------|
| Inherit | 0     | Inherit flag's value from map's parent. |
| Allow   | 1     | Allow saving.                           |
| Forbid  | 2     | Forbid saving.                          |

#### Encounters Tag
This tag describes the types of random enemy encounters possible within a map.

| Field          | Type   | Description                                         |
|:---------------|:-------|:----------------------------------------------------|
| TagID          | `EINT` | This will always be 41.                             |
| TagSize        | `EINT` | The size of the remaining fields measured in bytes. |
| EncounterCount | `EINT` | The number of random encounters.                    |
| Encounters     | [Monster Group Tag](#monster-group-tag) [`EncounterCount`] | Array of random encounters. |

#### Encounter Steps Tag
This tag specifies the number of steps between, or rarity of, random encounters.

| Field          | Type   | Description                                     |
|:---------------|:-------|:------------------------------------------------|
| TagID          | `EINT` | This will always be 44.                         |
| TagSize        | `EINT` | The size of `EncounterSteps` measured in bytes. |
| EncounterSteps | `EINT` | The number of steps between random encounters.  |

#### Area Rectangle Tag
This tag specifies the boundaries of a map's area.
This tag only affects maps whose `MapType` is `Area`.

The coordinates are measured in pixels and begin in the top-left corner of a map.
For example, an area rectangle of [0, 0, 100, 100] would completely cover a 100x100 map,
whereas [0, 0, 50, 50] would only cover the top-left quarter of said map.

| Field       | Type   | Description                                                            |
|:------------|:-------|:-----------------------------------------------------------------------|
| TagID       | `EINT` | This will always be 51.                                                |
| TagSize     | `EINT` | The size of the remaining fields measured in bytes. This should be 16. |
| LeftCoord   | `U32`  | The left-coordinate of the area rectangle.                             |
| TopCoord    | `U32`  | The top-coordinate of the area rectangle.                              |
| RightCoord  | `U32`  | The right-coordinate of the area rectangle.                            |
| BottomCoord | `U32`  | The bottom-coordinate of the area rectangle.                           |

### Map Start Tags
These tags are used by the [Map Start Structure](#map-start-structure).

#### Party Map ID Tag
This tag specifies the ID of the map the player should initially spawn in when starting a new game.

| Field    | Type   | Description                                             |
|:---------|:-------|:--------------------------------------------------------|
| TagID    | `EINT` | This will always be 1.                                  |
| TagSize  | `EINT` | The size of `PartyMap` measured in bytes.               |
| PartyMap | `EINT` | The ID of the map the player should initially spawn in. |

#### Party X Tag
This tag specifies the x-position the player should initially spawn at when starting a new game.

| Field   | Type   | Description                                          |
|:--------|:-------|:-----------------------------------------------------|
| TagID   | `EINT` | This will always be 2.                               |
| TagSize | `EINT` | The size of `PartyX` measured in bytes.              |
| PartyX  | `EINT` | The x-position the player should initially spawn at. |

`PartyX` is measured in map tiles.

#### Party Y Tag
This tag specifies the y-position the player should initially spawn at when starting a new game.

| Field   | Type   | Description                                          |
|:--------|:-------|:-----------------------------------------------------|
| TagID   | `EINT` | This will always be 3.                               |
| TagSize | `EINT` | The size of `PartyY` measured in bytes.              |
| PartyY  | `EINT` | The y-position the player should initially spawn at. |

`PartyY` is measured in map tiles.

#### Skiff Map ID Tag
This tag specifies the ID of the map the skiff vehicle should initially spawn in when starting a new game.

| Field    | Type   | Description                                                    |
|:---------|:-------|:---------------------------------------------------------------|
| TagID    | `EINT` | This will always be 11.                                        |
| TagSize  | `EINT` | The size of `SkiffMap` measured in bytes.                      |
| SkiffMap | `EINT` | The ID of the map the skiff vehicle should initially spawn in. |

#### Skiff X Tag
This tag specifies the x-position the skiff vehicle should initially spawn at when starting a new game.

| Field   | Type   | Description                                                 |
|:--------|:-------|:------------------------------------------------------------|
| TagID   | `EINT` | This will always be 12.                                     |
| TagSize | `EINT` | The size of `SkiffX` measured in bytes.                     |
| SkiffX  | `EINT` | The x-position the skiff vehicle should initially spawn at. |

`SkiffX` is measured in map tiles.

#### Skiff Y Tag
This tag specifies the y-position the skiff vehicle should initially spawn at when starting a new game.

| Field   | Type   | Description                                                 |
|:--------|:-------|:------------------------------------------------------------|
| TagID   | `EINT` | This will always be 13.                                     |
| TagSize | `EINT` | The size of `SkiffY` measured in bytes.                     |
| SkiffY  | `EINT` | The y-position the skiff vehicle should initially spawn at. |

`SkiffY` is measured in map tiles.

#### Ship Map ID Tag
This tag specifies the ID of the map the ship vehicle should initially spawn in when starting a new game.

| Field   | Type   | Description                                                   |
|:--------|:-------|:--------------------------------------------------------------|
| TagID   | `EINT` | This will always be 21.                                       |
| TagSize | `EINT` | The size of `ShipMap` measured in bytes.                      |
| ShipMap | `EINT` | The ID of the map the ship vehicle should initially spawn in. |

#### Ship X Tag
This tag specifies the x-position the ship vehicle should initially spawn at when starting a new game.

| Field   | Type   | Description                                                |
|:--------|:-------|:-----------------------------------------------------------|
| TagID   | `EINT` | This will always be 22.                                    |
| TagSize | `EINT` | The size of `ShipX` measured in bytes.                     |
| ShipX   | `EINT` | The x-position the ship vehicle should initially spawn at. |

`ShipX` is measured in map tiles.

#### Ship Y Tag
This tag specifies the y-position the ship vehicle should initially spawn at when starting a new game.

| Field   | Type   | Description                                                |
|:--------|:-------|:-----------------------------------------------------------|
| TagID   | `EINT` | This will always be 23.                                    |
| TagSize | `EINT` | The size of `ShipY` measured in bytes.                     |
| ShipY   | `EINT` | The y-position the ship vehicle should initially spawn at. |

`ShipY` is measured in map tiles.

#### Airship Map ID Tag
This tag specifies the ID of the map the airship vehicle should initially spawn in when starting a new game.

| Field      | Type   | Description                                                      |
|:-----------|:-------|:-----------------------------------------------------------------|
| TagID      | `EINT` | This will always be 31.                                          |
| TagSize    | `EINT` | The size of `AirshipMap` measured in bytes.                      |
| AirshipMap | `EINT` | The ID of the map the airship vehicle should initially spawn in. |

#### Airship X Tag
This tag specifies the x-position the airship vehicle should initially spawn at when starting a new game.

| Field    | Type   | Description                                                   |
|:---------|:-------|:--------------------------------------------------------------|
| TagID    | `EINT` | This will always be 32.                                       |
| TagSize  | `EINT` | The size of `AirshipX` measured in bytes.                     |
| AirshipX | `EINT` | The x-position the airship vehicle should initially spawn at. |

`AirshipX` is measured in map tiles.

#### Airship Y Tag
This tag specifies the y-position the airship vehicle should initially spawn at when starting a new game.

| Field    | Type   | Description                                                   |
|:---------|:-------|:--------------------------------------------------------------|
| TagID    | `EINT` | This will always be 33.                                       |
| TagSize  | `EINT` | The size of `AirshipY` measured in bytes.                     |
| AirshipY | `EINT` | The y-position the airship vehicle should initially spawn at. |

`AirshipY` is measured in map tiles.

### Music Tags
These tags are used by the [Music Tag](#music-tag).

#### Music Name Tag
This tag provides the filename of a song.

| Field     | Type          | Description                                  |
|:----------|:--------------|:---------------------------------------------|
| TagID     | `EINT`        | This will always be 1.                       |
| TagSize   | `EINT`        | The number of characters in `MusicName`.     |
| MusicName | `U8[TagSize]` | The characters that make up the song's name. |

If `MusicName` is "(OFF)" or an empty string, then the song is considered empty and no song should play.

`TagSize` + `MusicName` essentially acts as a [`STRING`](#string-type) type.

#### Music Fade Time Tag
This tag provides the amount of time a song should fade-in for, measured in milliseconds.

| Field    | Type   | Description                                             |
|:---------|:-------|:--------------------------------------------------------|
| TagID    | `EINT` | This will always be 2.                                  |
| TagSize  | `EINT` | The size of `FadeTime` measured in bytes.               |
| FadeTime | `EINT` | The number of milliseconds the song should fade in for. |

The value of `FadeTime` ranges from 0 ms to 10000 ms (i.e. 0 to 10 seconds).

#### Music Volume Tag
This tag specifies the volume a song should be played at.

| Field   | Type   | Description                             |
|:--------|:-------|:----------------------------------------|
| TagID   | `EINT` | This will always be 3.                  |
| TagSize | `EINT` | The size of `Volume` measured in bytes. |
| Volume  | `EINT` | The volume the song should play at.     |

`Volume` is an integer percentage and ranges from 0% to 100%.

#### Music Tempo Tag
This tag specifies the tempo a song should be played at.

| Field   | Type   | Description                            |
|:--------|:-------|:---------------------------------------|
| TagID   | `EINT` | This will always be 4.                 |
| TagSize | `EINT` | The size of `Tempo` measured in bytes. |
| Tempo   | `EINT` | The tempo the song should play at.     |

`Tempo` is an integer percentage and ranges from 0% to 150%.

#### Music Balance Tag
This tag specifies the left-right balance a song should be played with.

| Field   | Type   | Description                                       |
|:--------|:-------|:--------------------------------------------------|
| TagID   | `EINT` | This will always be 5.                            |
| TagSize | `EINT` | The size of `Balance` measured in bytes.          |
| Balance | `EINT` | The left-right balance the song should play with. |

`Balance` is an integer that ranges from 0 to 100.
A value of 50 is centered, 0 is left-balance only, and 100 is right-balance only.

### Encounter Tags
