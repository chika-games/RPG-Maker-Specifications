# LCF Map Tree Tags
Below are the various tags used exclusively by LCF Map Tree (LMT) files.

For more information on tags, please consult the [LCF page](lcf.md).

## Map Info Tags
These tags are used within the context of [Map Info Structures](lmt.md#map-info-structure).

### Map Name Tag
This tag specifies the name of a map.

| Field   | Type          | Description                                 |
|:--------|:--------------|:--------------------------------------------|
| TagID   | `EINT`        | This will always be 1.                      |
| TagSize | `EINT`        | The number of characters in `MapName`.      |
| MapName | `U8[TagSize]` | The characters that make up the map's name. |

`TagSize` + `MapName` make a [`STRING`](#string-type).

### Parent ID Tag
This tag provides the ID of a map's parent.

| Field    | Type   | Description                               |
|:---------|:-------|:------------------------------------------|
| TagID    | `EINT` | This will always be 2.                    |
| TagSize  | `EINT` | The size of `ParentID` measured in bytes. |
| ParentID | `EINT` | The ID of the parent map.                 |

### Indentation Tag
This tag specifies a map's indentation in the LMT [Map Hierarchy](lmt.md#map-hierarchy).

| Field       | Type          | Description                                  |
|:------------|:--------------|:---------------------------------------------|
| TagID       | `EINT`        | This will always be 3.                       |
| TagSize     | `EINT`        | The size of `Indentation` measured in bytes. |
| Indentation | `EINT`        | The indentation of the map.                  |

### Map Type Tag
This tag specifies a map's type.

| Field   | Type   | Description                                      |
|:--------|:-------|:-------------------------------------------------|
| TagID   | `EINT` | This will always be 4.                           |
| TagSize | `EINT` | The size of `MapType` measured in bytes.         |
| MapType | `EINT` | The map's type. See [Map Type](#map-type) below. |

#### Map Type
The possible values of `MapType`.

| Type | Value | Description       |
|:-----|:------|:------------------|
| Root | 0     | The root map.     |
| Map  | 1     | An ordinary map.  |
| Area | 2     | An area of a map. |

### Edit Position X Tag
This specifies the RM2k/3 Editor's x-position when a particular map was last edited.

| Field    | Type   | Description                               |
|:---------|:-------|:------------------------------------------|
| TagID    | `EINT` | This will always be 5.                    |
| TagSize  | `EINT` | The size of `EditPosX` measured in bytes. |
| EditPosX | `EINT` | The editor's last x-position.             |

### Edit Position Y Tag
This specifies the RM2k/3 Editor's y-position when a particular map was last edited.

| Field    | Type   | Description                               |
|:---------|:-------|:------------------------------------------|
| TagID    | `EINT` | This will always be 6.                    |
| TagSize  | `EINT` | The size of `EditPosY` measured in bytes. |
| EditPosY | `EINT` | The editor's last y-position.             |

### Edit Expanded Tag
This specifies whether or not a map was "expanded" within the the RM2k/3 editor;
i.e., it's children maps were visible in the [Map Hierarchy](lmt.md#map-hierarchy).

| Field        | Type   | Description                                    |
|:-------------|:-------|:-----------------------------------------------|
| TagID        | `EINT` | This will always be 7.                         |
| TagSize      | `EINT` | The size of `EditExpanded` measured in bytes.  |
| EditExpanded | `EINT` | Whether or not the map was expanded in-editor. |

### Music Type Tag
This tag specifies how music should be played within a map.

| Field     | Type   | Description                                                |
|:----------|:-------|:-----------------------------------------------------------|
| TagID     | `EINT` | This will always be 11.                                    |
| TagSize   | `EINT` | The size of `MusicType` measured in bytes.                 |
| MusicType | `EINT` | The map's music type. See [Music Type](#music-type) below. |

#### Music Type
The possible values of `MusicType`.

| Type      | Value | Description                           |
|:----------|:------|:--------------------------------------|
| Inherit   | 0     | Inherit music type from map's parent. |
| Event     | 1     | Music provided through events.        |
| Specified | 2     | Play a specific song.                 |

### Music Tag
This tag specifies a particular song and its playback settings.

Not all fields may be present; the specified default value should be used in place of missing fields.

| Field    | Type                                        | Default Value   | Description                                               |
|:---------|:--------------------------------------------|:----------------|:----------------------------------------------------------|
| TagID    | `EINT`                                      | Always present. | This will always be 12.                                   |
| TagSize  | `EINT`                                      | Always present. | The total size of the remaining fields measured in bytes. |
| Name     | [Music Name Tag](#music-name-tag)           | `(OFF)`         | The filename of the song to play.                         |
| FadeTime | [Music Fade Time Tag](#music-fade-time-tag) | 0               | The fade time for the music; 0 means no fade.             |
| Volume   | [Music Volume Tag](#music-volume-tag)       | 100             | The volume of the music.                                  |
| Tempo    | [Music Tempo Tag](#music-tempo-tag)         | 100             | The tempo of the music.                                   |
| Balance  | [Music Balance Tag](#music-balance-tag)     | 50              | The left-right balance of the music; 50 is centered.      |
| End      | [End Tag](common_tags.md#end-tag)           | Always present. | Indicates the end of the music tag.                       |

### Background Type Tag
This tag specifies the type of background a map has.

| Field          | Type   | Description                                                               |
|:---------------|:-------|:--------------------------------------------------------------------------|
| TagID          | `EINT` | This will always be 21.                                                   |
| TagSize        | `EINT` | The size of `BackgroundType` measured in bytes.                           |
| BackgroundType | `EINT` | The map's background type. See [Background Type](#background-type) below. |

#### Background Type

| Type      | Value | Description                           |
|:----------|:------|:--------------------------------------|
| Inherit   | 0     | Inherit music type from map's parent. |
| Terrain   | 1     | Use the terrain settings.             |
| Specified | 2     | Use a specified background image.     |

### Background Name Tag
This tag provides the filename of a map's background image.

| Field          | Type          | Description                                        |
|:---------------|:--------------|:---------------------------------------------------|
| TagID          | `EINT`        | This will always be 22.                            |
| TagSize        | `EINT`        | The number of characters in `BackgroundName`.      |
| BackgroundName | `U8[TagSize]` | The characters that make up the background's name. |

`TagSize` + `BackgroundName` make a [`STRING`](#string-type).

### Teleport Flag Tag
This tag specifies whether or not teleportation is allowed within a map.

| Field        | Type   | Description                                   |
|:-------------|:-------|:----------------------------------------------|
| TagID        | `EINT` | This will always be 31.                       |
| TagSize      | `EINT` | The size of `TeleportFlag` measured in bytes. |
| TeleportFlag | `FLAG` | Whether or not teleportation is allowed.      |

### Escape Flag Tag
This tag specifies whether or not escaping is allowed within a map.

| Field      | Type   | Description                                 |
|:-----------|:-------|:--------------------------------------------|
| TagID      | `EINT` | This will always be 32.                     |
| TagSize    | `EINT` | The size of `EscapeFlag` measured in bytes. |
| EscapeFlag | `FLAG` | Whether or not escaping is allowed.         |

### Save Flag Tag
This tag specifies whether or not saving is allowed within a map.

| Field    | Type   | Description                               |
|:---------|:-------|:------------------------------------------|
| TagID    | `EINT` | This will always be 33.                   |
| TagSize  | `EINT` | The size of `SaveFlag` measured in bytes. |
| SaveFlag | `FLAG` | Whether or not saving is allowed.         |

### Encounters Tag
This tag describes the possible random enemy encounters within a map.

| Field          | Type                                           | Description                                         |
|:---------------|:-----------------------------------------------|:----------------------------------------------------|
| TagID          | `EINT`                                         | This will always be 41.                             |
| TagSize        | `EINT`                                         | The size of the remaining fields measured in bytes. |
| EncounterCount | `EINT`                                         | The number of random encounters in `Encounters`.    |
| Encounters     | `LIST` [Monster Group Tag](#monster-group-tag) | List of possible random encounters.                 |

### Encounter Steps Tag
This tag specifies the number of steps between random encounters (i.e. their rarity).

| Field          | Type   | Description                                     |
|:---------------|:-------|:------------------------------------------------|
| TagID          | `EINT` | This will always be 44.                         |
| TagSize        | `EINT` | The size of `EncounterSteps` measured in bytes. |
| EncounterSteps | `EINT` | The number of steps between random encounters.  |

### Area Rectangle Tag
This tag specifies the boundaries of a map's area. This tag only applies to maps whose `MapType` is `Area`.

The coordinates are measured in pixels, and the origin is in the top-left corner of a map.
For example, an area rectangle of [0, 0, 100, 100] would completely cover a 100x100 map,
but [0, 0, 50, 50] would only cover the top-left quarter of a 100x100 map.

| Field       | Type   | Description                                         |
|:------------|:-------|:----------------------------------------------------|
| TagID       | `EINT` | This will always be 51.                             |
| TagSize     | `EINT` | The size of the remaining fields measured in bytes. |
| LeftCoord   | `U32`  | The left-coordinate of the area rectangle.          |
| TopCoord    | `U32`  | The top-coordinate of the area rectangle.           |
| RightCoord  | `U32`  | The right-coordinate of the area rectangle.         |
| BottomCoord | `U32`  | The bottom-coordinate of the area rectangle.        |

## Map Start Tags
These tags are used within the context of [Map Start Structures](lmt.md#map-start-structure).

### Party Map ID Tag
This tag specifies the ID of the map the player should initially spawn in when starting a new game.

| Field    | Type   | Description                                             |
|:---------|:-------|:--------------------------------------------------------|
| TagID    | `EINT` | This will always be 1.                                  |
| TagSize  | `EINT` | The size of `PartyMap` measured in bytes.               |
| PartyMap | `EINT` | The ID of the map the player should initially spawn in. |

### Party X Tag
This tag specifies the x-position the player should initially spawn at when starting a new game.

| Field   | Type   | Description                                          |
|:--------|:-------|:-----------------------------------------------------|
| TagID   | `EINT` | This will always be 2.                               |
| TagSize | `EINT` | The size of `PartyX` measured in bytes.              |
| PartyX  | `EINT` | The x-position the player should initially spawn at. |

`PartyX` is measured in map tiles.

### Party Y Tag
This tag specifies the y-position the player should initially spawn at when starting a new game.

| Field   | Type   | Description                                          |
|:--------|:-------|:-----------------------------------------------------|
| TagID   | `EINT` | This will always be 3.                               |
| TagSize | `EINT` | The size of `PartyY` measured in bytes.              |
| PartyY  | `EINT` | The y-position the player should initially spawn at. |

`PartyY` is measured in map tiles.

### Skiff Map ID Tag
This tag specifies the ID of the map the skiff vehicle should initially spawn in when starting a new game.

| Field    | Type   | Description                                                    |
|:---------|:-------|:---------------------------------------------------------------|
| TagID    | `EINT` | This will always be 11.                                        |
| TagSize  | `EINT` | The size of `SkiffMap` measured in bytes.                      |
| SkiffMap | `EINT` | The ID of the map the skiff vehicle should initially spawn in. |

### Skiff X Tag
This tag specifies the x-position the skiff vehicle should initially spawn at when starting a new game.

| Field   | Type   | Description                                                 |
|:--------|:-------|:------------------------------------------------------------|
| TagID   | `EINT` | This will always be 12.                                     |
| TagSize | `EINT` | The size of `SkiffX` measured in bytes.                     |
| SkiffX  | `EINT` | The x-position the skiff vehicle should initially spawn at. |

`SkiffX` is measured in map tiles.

### Skiff Y Tag
This tag specifies the y-position the skiff vehicle should initially spawn at when starting a new game.

| Field   | Type   | Description                                                 |
|:--------|:-------|:------------------------------------------------------------|
| TagID   | `EINT` | This will always be 13.                                     |
| TagSize | `EINT` | The size of `SkiffY` measured in bytes.                     |
| SkiffY  | `EINT` | The y-position the skiff vehicle should initially spawn at. |

`SkiffY` is measured in map tiles.

### Ship Map ID Tag
This tag specifies the ID of the map the ship vehicle should initially spawn in when starting a new game.

| Field   | Type   | Description                                                   |
|:--------|:-------|:--------------------------------------------------------------|
| TagID   | `EINT` | This will always be 21.                                       |
| TagSize | `EINT` | The size of `ShipMap` measured in bytes.                      |
| ShipMap | `EINT` | The ID of the map the ship vehicle should initially spawn in. |

### Ship X Tag
This tag specifies the x-position the ship vehicle should initially spawn at when starting a new game.

| Field   | Type   | Description                                                |
|:--------|:-------|:-----------------------------------------------------------|
| TagID   | `EINT` | This will always be 22.                                    |
| TagSize | `EINT` | The size of `ShipX` measured in bytes.                     |
| ShipX   | `EINT` | The x-position the ship vehicle should initially spawn at. |

`ShipX` is measured in map tiles.

### Ship Y Tag
This tag specifies the y-position the ship vehicle should initially spawn at when starting a new game.

| Field   | Type   | Description                                                |
|:--------|:-------|:-----------------------------------------------------------|
| TagID   | `EINT` | This will always be 23.                                    |
| TagSize | `EINT` | The size of `ShipY` measured in bytes.                     |
| ShipY   | `EINT` | The y-position the ship vehicle should initially spawn at. |

`ShipY` is measured in map tiles.

### Airship Map ID Tag
This tag specifies the ID of the map the airship vehicle should initially spawn in when starting a new game.

| Field      | Type   | Description                                                      |
|:-----------|:-------|:-----------------------------------------------------------------|
| TagID      | `EINT` | This will always be 31.                                          |
| TagSize    | `EINT` | The size of `AirshipMap` measured in bytes.                      |
| AirshipMap | `EINT` | The ID of the map the airship vehicle should initially spawn in. |

### Airship X Tag
This tag specifies the x-position the airship vehicle should initially spawn at when starting a new game.

| Field    | Type   | Description                                                   |
|:---------|:-------|:--------------------------------------------------------------|
| TagID    | `EINT` | This will always be 32.                                       |
| TagSize  | `EINT` | The size of `AirshipX` measured in bytes.                     |
| AirshipX | `EINT` | The x-position the airship vehicle should initially spawn at. |

`AirshipX` is measured in map tiles.

### Airship Y Tag
This tag specifies the y-position the airship vehicle should initially spawn at when starting a new game.

| Field    | Type   | Description                                                   |
|:---------|:-------|:--------------------------------------------------------------|
| TagID    | `EINT` | This will always be 33.                                       |
| TagSize  | `EINT` | The size of `AirshipY` measured in bytes.                     |
| AirshipY | `EINT` | The y-position the airship vehicle should initially spawn at. |

`AirshipY` is measured in map tiles.

## Music Tags
These tags are used within the context of [Music Tags](#music-tag).

### Music Name Tag
This tag provides the filename of a song.

| Field     | Type          | Description                                  |
|:----------|:--------------|:---------------------------------------------|
| TagID     | `EINT`        | This will always be 1.                       |
| TagSize   | `EINT`        | The number of characters in `MusicName`.     |
| MusicName | `U8[TagSize]` | The characters that make up the song's name. |

If `MusicName` is `(OFF)` or an empty string, then the song is considered empty and no song should play.

`TagSize` + `MusicName` make a [`STRING`](#string-type).

### Music Fade Time Tag
This tag provides the amount of time a song should fade-in for, measured in milliseconds.

| Field    | Type   | Description                                             |
|:---------|:-------|:--------------------------------------------------------|
| TagID    | `EINT` | This will always be 2.                                  |
| TagSize  | `EINT` | The size of `FadeTime` measured in bytes.               |
| FadeTime | `EINT` | The number of milliseconds the song should fade in for. |

The value of `FadeTime` ranges from 0 ms to 10000 ms (i.e. 0 to 10 seconds).

### Music Volume Tag
This tag specifies the volume a song should be played at.

| Field   | Type   | Description                             |
|:--------|:-------|:----------------------------------------|
| TagID   | `EINT` | This will always be 3.                  |
| TagSize | `EINT` | The size of `Volume` measured in bytes. |
| Volume  | `EINT` | The volume the song should play at.     |

`Volume` is an integer percentage and ranges from 0% to 100%.

### Music Tempo Tag
This tag specifies the tempo a song should be played at.

| Field   | Type   | Description                            |
|:--------|:-------|:---------------------------------------|
| TagID   | `EINT` | This will always be 4.                 |
| TagSize | `EINT` | The size of `Tempo` measured in bytes. |
| Tempo   | `EINT` | The tempo the song should play at.     |

`Tempo` is an integer percentage and ranges from 0% to 150%.

### Music Balance Tag
This tag specifies the left-right balance a song should be played with.

| Field   | Type   | Description                                       |
|:--------|:-------|:--------------------------------------------------|
| TagID   | `EINT` | This will always be 5.                            |
| TagSize | `EINT` | The size of `Balance` measured in bytes.          |
| Balance | `EINT` | The left-right balance the song should play with. |

`Balance` is an integer that ranges from 0 to 100.
A value of 50 is centered, 0 is left-balance only, and 100 is right-balance only.

### Encounter Tags
These tags are used within the context of [Encounters Tags](#encounters-tag).

#### Monster Group Tag
This tag provides the ID of a monster group to be used in a random encounter.

| Field   | Type   | Description                                                                          |
|:--------|:-------|:-------------------------------------------------------------------------------------|
| TagID   | `EINT` | This will always be 1.                                                               |
| TagSize | `EINT` | The size of `GroupID` measured in bytes.                                             |
| GroupID | `EINT` | The ID of the monster group that will be attacking in a particular random encounter. |
