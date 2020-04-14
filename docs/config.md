---
layout: default
---

# Configuration File Specification
## Introduction
The `RPG_RT.ini` configuation file provides basic start-up information for RPG Maker 2000/2003 games.
This file in [INI](https://en.wikipedia.org/wiki/INI_file) format.

This file is stored within the same directory as the RPG Maker runtime (`RPG_RT.exe`) but is not required:
if the file is not present or fields are omitted, the runtime will use specific default values instead.

## Data Types
The following tables list of the data types that will be used within this specification.

| Type     | Description                                 |
|:---------|:--------------------------------------------|
| `STRING` | A string of characters.                   |
| `UINT`   | An unsigned integer.                      |
| `BOOL`   | A boolean integer: 0 is false, 1 is true. |

`STRING` has no standard encoding; the current system locale is used by the runtime.
For example, a string encoded in [Shift_JIS](https://en.wikipedia.org/wiki/Shift_JIS) may appear incorrectly if the runtime is running on a machine with an English locale.

This is likely due to RPG Maker 2000/2003 originally being a Japanese-only engine; Shift_JIS was likely assumed to be the encoding of choice.
This has resulted in various different encodings being used across RPG Maker 2000/2003 games, although UTF-8 has become dominant for modern games.

## Config File Structure
`RPG_RT.ini` is stored in textual form according to the INI format.

The following table describes the overall structure of the configuration file.

| Field           | Type     | Default Value | Description                       |
|:----------------|:---------|:--------------|:----------------------------------|
| GameTitle       | `STRING` | "Untitled"    | The title of the game's window.   |
| MapEditMode     | `UINT`   | 0             | The last layer-editing mode used. |
| MapEditZoom     | `UINT`   | 0             | The last zoom level used.         |
| FullPackageFlag | `BOOL`   | 0 (false)     | Skip loading a runtime package.   |

If a field is missing or has an invalid value, then the runtime will use the field's default value.

`GameTitle` determines the title of the runtime's window while playing the game.

`MapEditMode` indicates the last layer-editing mode used when editing the game in the engine's editor.

`MapEditZoom` indicates the last zoom level used when editing the game in the engine's editor.

`FullPackageFlag` determines whether or not the runtime should attempt to load a runtime package before running the game.
Such runtime packages are installed on the player's machine and provide default assets for games to use;
games that make use of a runtime package will not work properly if the package is not installed on the user's machine and loaded by the runtime.
This is done to keep games from distributing the same default assets, potentially reducing the amount of disk-space games take up.

If a game does not make use of a runtime package, then `FullPackageFlag` may be set to 1 to allow the runtime to skip loading
a runtime package, thereby improving startup performance and not requiring the player to install one.

The official runtime packages for RPG Maker 2000 and RPG Maker 2003 can obtained from the [official website](https://www.rpgmakerweb.com/download/additional/run-time-packages).
