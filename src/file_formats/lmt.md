# LCF Map Tree File Specification
## Introduction
LCF Map Tree (LMT) files are used to store map properties, game start information, and map orderings for RM2k/3 games.

RM2k/3 games have a single LMT file: `RPG_RT.lmt`. This should always be located within the same directory as the RM2k/3 Runtime (i.e. `RPG_RT.exe`).
This file is used in both the RM2k/3 Editors and Runtimes.

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
| MapStart      | [Map Start](#map-start-structure)               | Structure containing game start information.              |

All maps are arranged in a hierarchy and are children to the first map in the `MapInfos` array called the root map.
This root map is not meant to be playable and merely acts as the default parent to all top-level maps in the hierarchy.
In older versions of the RM2k/3 Runtimes, the name of the root map was used to determine the name of the Runtime's window.
This functionality has since been deprecated in favor of a [configuration file](config.md).

`ActiveNode` is used by the RM2k/3 Editors to keep track of the last saved/edited map.
This allows the Editor to automatically open the last edited map on startup.

## Map Info Structure
The following table describes the layout of `Map Info` structures.

**Note:** In practice, not all of the following tag-based fields will be present. This is presumably to reduce the overall size of LMT files.
If a field is missing, then its specified default value should be used.

| Field          | Type                                                   | Default Value                                               | Description                                                                                                                    |
|:---------------|:-------------------------------------------------------|:------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------|
| MapID          | `EINT`                                                 | Always present.                                             | The map's unique ID. `0` is always the root map's ID.                                                                          |
| MapName        | [Map Name Tag](lmt_tags.md#map-name-tag)               | Always present.                                             | The map's name.                                                                                                                |
| ParentID       | [Parent ID Tag](lmt_tags.md#parent-id-tag)             | 0                                                           | The ID of a map's parent. `0` means it's a top-level map (root is parent).                                                     |
| Indentation    | [Indentation Tag](lmt_tags.md#indentation-tag)         | 1 (0 if root)                                               | The map's hierarchical indentation; it indicates how deep it is in the hierarchy. See the [example hierarchy](#map-hierarchy). |
| MapType        | [Map Type Tag](lmt_tags.md#map-type-tag)               | Map (1)                                                     | The type of map.                                                                                                               |
| EditPosX       | [Edit Position X Tag](lmt_tags.md#edit-position-x-tag) | 0                                                           | The editor's last x-position when the map was last edited.                                                                     |
| EditPosY       | [Edit Position Y Tag](lmt_tags.md#edit-position-x-tag) | 0                                                           | The editor's last y-position when the map was last edited.                                                                     |
| EditExpanded   | [Edit Expanded Tag](lmt_tags.md#edit-expanded-tag)     | False (0)                                                   | Whether or not the map was expanded in the editor's tree view (children were visible).                                         |
| MusicType      | [Music Type Tag](common_tags.md#music-type-tag)        | Event (1) for top-level maps; Inherit (0) for child maps.   | How map's background music should be played.                                                                                   |
| Music          | [Music Tag](common_tags.md#music-tag)                  | See tag's section.                                          | The map's background music and its playback settings.                                                                          |
| BackgroundType | [Background Type Tag](lmt_tags.md#background-type-tag) | Terrain (1) for top-level maps; Inherit (0) for child maps. | The type of background to display while in combat.                                                                             |
| BackgroundName | [Background Name Tag](lmt_tags.md#background-name-tag) | `backdrop` or an empty string                               | The filename of the combat background.                                                                                         |
| TeleportFlag   | [Teleport Flag Tag](lmt_tags.md#teleport-flag-tag)     | Allow (1) for top-level maps; Inherit (0) for child maps.   | Determines whether or not teleporting out of the map is allowed.                                                               |
| EscapeFlag     | [Escape Flag Tag](lmt_tags.md#escape-flag-tag)         | Allow (1) for top-level maps; Inherit (0) for child maps.   | Determines whether or not escaping out of the map is allowed.                                                                  |
| SaveFlag       | [Save Flag Tag](lmt_tags.md#save-flag-tag)             | Allow (1) for top-level maps; Inherit (0) for child maps.   | Determines whether or not saving is allowed within the map.                                                                    |
| Encounters     | [Encounters Tag](lmt_tags.md#encounters-tag)           | No random encounters; e.g. an empty array/list.             | All possible (random) combat encounters that can appear within the map.                                                        |
| EncounterSteps | [Encounter Steps Tag](lmt_tags.md#encounter-steps-tag) | 25                                                          | The likelihood of having a random encounter.                                                                                   |
| AreaRectangle  | [Area Rectangle Tag](lmt_tags.md#area-rectangle-tag)   | [0, 0, 0, 0]                                                | The size of a map area measured in pixels. |
| End            | [End Tag](common_tags.md#end-tag)                      | Always present.                                             | Indicates the end of the structure. |

### Map Hierarchy
All maps of an RM2k/3 game are arranged in a parent-child hierarchy where the root map is at the top of the hierarchy.
Below is an example of such a hierarchy that is intended to help illustrate some of the fields of the `Map Info` structure.

```text
. My Game (root; indentation=0)
+-- Map 1 (parent=0; indentation=1)
|   +-- Map 2 (parent=1; indentation=2)
|   |   +-- Map 5 (parent=2; indentation=3)
|   +-- Map 4 (parent=1; indentation=2)
+-- Map 3 (parent=0; indentation=1)
```

Notice that the maps are not required to be in sequential order. Map orderings are determined by the `MapOrders` field.
Additionally, the ordering of the maps are mainly used by the editors and don't necessarily reflect the actual in-game order.

## Map Start Structure
The following table describes the layout of the `Map Start` structure.
This structure specifies the maps and positions the player and vehicles should spawn at when a new game is created.

All x and y positions are measured in map tiles.

**Note:** In practice, not all of this structure's tag-based fields will be present.
This can happen when the player or vehicles were not given a starting position within the game (i.e. they're not used).
If the player is not assigned a starting position, the RM2k/3 Runtime will raise an error when attempting to play.

| Field        | Type                                                 | Description                                                                                                                    |
|:-------------|:-----------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------|
| PartyMapID   | [Party Map ID Tag](lmt_tags.md#party-map-id-tag)     | The ID of the map the player should initially spawn in.          |
| PartyX       | [Party X Tag](lmt_tags.md#party-x-tag)               | The x-position the player should spawn at in their starting map. |
| PartyY       | [Party Y Tag](lmt_tags.md#party-y-tag)               | The y-position the player should spawn at in their starting map. |
| SkiffMapID   | [Skiff Map ID Tag](lmt_tags.md#skiff-map-id-tag)     | The ID of the map the skiff vehicle should initially spawn in.   |
| SkiffX       | [Skiff X Tag](lmt_tags.md#skiff-x-tag)               | The x-position the skiff should spawn at in its starting map.    |
| SkiffY       | [Skiff Y Tag](lmt_tags.md#skiff-y-tag)               | The y-position the skiff should spawn at in its starting map.    |
| ShipMapID    | [Ship Map ID Tag](lmt_tags.md#ship-map-id-tag)       | The ID of the map the ship vehicle should initially spawn in.    |
| ShipX        | [Ship X Tag](lmt_tags.md#ship-x-tag)                 | The x-position the ship should spawn at in its starting map.     |
| ShipY        | [Ship Y Tag](lmt_tags.md#ship-y-tag)                 | The y-position the ship should spawn at in its starting map.     |
| AirshipMapID | [Airship Map ID Tag](lmt_tags.md#airship-map-id-tag) | The ID of the map the airship vehicle should initially spawn in. |
| AirshipX     | [Airship X Tag](lmt_tags.md#airship-x-tag)           | The x-position the airship should spawn at in its starting map.  |
| AirshipY     | [Airship Y Tag](lmt_tags.md#airship-y-tag)           | The y-position the airship should spawn at in its starting map.  |
| End          | [End Tag](common_tags.md#end-tag)                    | Indicates the end of the structure.                              |
