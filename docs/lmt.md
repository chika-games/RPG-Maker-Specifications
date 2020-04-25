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
| ActiveNode    | `EINT`                                          | The ID of the last active/edited map.                     |
| MapStart      | [Map Start](#map-start-structure)               | Game start information, such as starting positions.       |

All maps are arranged in a hierarchy and are children to the first map in the `MapInfos` array called the root map.
This root map is not meant to be playable and is simply meant to be the default parent to all maps in the hierarchy.
In older versions of the RM2k/3 runtimes, the name of the root map was also used to determine the name of the game's window.
This functionality has since been deprecated in favor of a [configuration file](./config.html).

`ActiveNode` is used by the RM2k/3 editors to keep track of the last edited map.
This allows them to automatically open the last edited map upon launching the editor.

## Map Info Structure
The following table describes the layout of `Map Info` structures.

| Field          | Type                                        | Default Value                                             | Description                                                                                                                    |
|:---------------|:--------------------------------------------|:----------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------|
| MapID          | `EINT`                                      | Always present.                                           | The map's unique ID. `0` is always the root map's ID.                                                                          |
| MapName        | [Map Name Tag](#map-name-tag)               | Always present.                                           | The map's name.                                                                                                                |
| ParentID       | [Parent ID Tag](#parent-id-tag)             | 0                                                         | The ID of a map's parent. `0` means it's a top-level map (root is parent).                                                     |
| Indentation    | [Indentation Tag](#indentation-tag)         | 1 (0 if root)                                             | The map's hierarchical indentation; it indicates how deep it is in the hierarchy. See the [example hierarchy](#map-hierarchy). |
| MapType        | [Map Type Tag](#map-type-tag)               | Map (1)                                                   | The type of map.                                                                                                               |
| EditPosX       | [Edit Position X Tag](#edit-position-x-tag) | 0                                                         | The editor's last x-position when the map was last edited.                                                                     |
| EditPosY       | [Edit Position Y Tag](#edit-position-x-tag) | 0                                                         | The editor's last y-position when the map was last edited.                                                                     |
| EditExpanded   | [Edit Expanded Tag](#edit-expanded-tag)     | False (0)                                                 | Whether or not the map was expanded in the editor's tree view (children were visible).                                         |
| MusicType      | [Music Type Tag](#music-type-tag)           | See tag's section.                                        | How map's background music should be played.                                                                                   |
| Music          | [Music Tag](#music-tag)                     | See tag's section.                                        | The map's background music and its playback settings.                                                                          |
| BackgroundType | [Background Type Tag](#background-type-tag) | See tag's section.                                        | The type of background to display while in combat.                                                                             |
| BackgroundName | [Background Name Tag](#background-name-tag) | "backdrop" or ""                                          | The filename of the combat background.                                                                                         |
| TeleportFlag   | [Teleport Flag Tag](#teleport-flag-tag)     | Allow (1) for top-level maps; Inherit (0) for child maps. | Determines whether or not teleporting out of the map is allowed.                                                               |
| EscapeFlag     | [Escape Flag Tag](#escape-flag-tag)         | Allow (1) for top-level maps; Inherit (0) for child maps. | Determines whether or not escaping out of the map is allowed.                                                                  |
| SaveFlag       | [Save Flag Tag](#save-flag-tag)             | Allow (1) for top-level maps; Inherit (0) for child maps. | Determines whether or not saving is allowed within the map.                                                                    |
| Encounters     | [Encounters Tag](#encounters-tag)           | An empty array.                                           | All possible (random) combat encounters that can appear within the map.                                                        |
| EncounterSteps | [Encounter Steps Tag](#encounter-steps-tag) | 25                                                        | The likelihood of having a random encounter.                                                                                   |
| AreaRectangle  | [Area Rectangle Tag](#area-rectangle-tag)   | [0, 0, 0, 0]                                              | The size of a map area measured in pixels. |
| End            | [End Tag](#end-tag)                         | Always present.                                           | Indicates the end of the map info structure. |

**Note:** In practice, not all of the above fields with a tag-based type will be present. This is to help reduce the overall size of LMT files.
If such a field is missing, then its respective default value provided above should be used in its place.

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
Marks the end of a structure or tag. This tag only has an ID field.

| Field | Type   | Description              |
|:------|:-------|:-------------------------|
| TagID | `EINT` | This will always be 0. |

### Map Info Tags
#### Map Name Tag
This tag provides the name of a map.

| Field   | Type          | Description                                 |
|:--------|:--------------|:--------------------------------------------|
| TagID   | `EINT`        | This will always be 1.                      |
| TagSize | `EINT`        | The number of characters in `MapName`.      |
| MapName | `U8[TagSize]` | The characters that make up the map's name. |

`TagSize` + `MapName` effectively act as a [`STRING`](#string-type) type.

#### Parent ID Tag
This tag provides the ID of a map's parent map.

| Field    | Type          | Description                                 |
|:---------|:--------------|:--------------------------------------------|
| TagID    | `EINT`        | This will always be 2.                      |
| TagSize  | `EINT`        | The size of `ParentID` measured in bytes.   |
| ParentID | `EINT`        | The ID of the map's parent.                 |

#### Indentation Tag
This tag provides a map's indentation in the (Map Hierarchy)[#map-hierarchy].

| Field       | Type          | Description                                  |
|:------------|:--------------|:---------------------------------------------|
| TagID       | `EINT`        | This will always be 3.                       |
| TagSize     | `EINT`        | The size of `Indentation` measured in bytes. |
| Indentation | `EINT`        | The indentation of the map.                  |

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

### Map Start Tags

### Music Tags
These tags are used within the [Music Tag](#music-tag).

#### Music Name Tag
This tag specifies the filename of a song.

| Field | Type   | Default Value   | Description     |
|:------|:-------|:----------------|:----------------|
| TagID | EINT   | This will always be 1.            |
| Name  | STRING | The filename of the song to play. |

If `Name` is "(OFF)" or an empty string, then no the song is considered empty; no song should play.

### Encounter Tags
