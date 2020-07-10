# Configuration File Specification
## Introduction
The `RPG_RT.ini` configuation file provides basic start-up information for RPG Maker 2000/2003 (RM2k/3) games
and is used by both the RM2k/3 editor and runtime: the editor will read this file on startup and modify it when closing,
and the runtime will only read it on startup.

This file is not required, but, if present, should always live within the same directory as the RM2k/3 runtime (`RPG_RT.exe`).
If the file is not present or fields are omitted, the RM2k/3 runtime will use specific default values when necessary.

## Configuration File Structure
`RPG_RT.ini` is a text file that follows the [INI](https://en.wikipedia.org/wiki/INI_file) format.

The following table describes the overall structure of the configuration file.
The fields are in no particular order and may be omitted.

| Field           | Type     | Default Value | Description                       |
|:----------------|:---------|:--------------|:----------------------------------|
| GameTitle       | `STRING` | `Untitled`    | The title of the game's window.   |
| MapEditMode     | `UINT`   | `0`           | The last layer-editing mode used. |
| MapEditZoom     | `UINT`   | `0`           | The last zoom level used.         |
| FullPackageFlag | `BOOL`   | `0`           | Skip loading a runtime package.   |

If a field is missing or has an invalid value, then the runtime will use the field's specified default value instead.

`GameTitle` determines the title of the runtime's window.

`MapEditMode` indicates the last layer-editing mode used when editing the game in the engine's editor. See [Map Edit Modes](#map-edit-modes).

`MapEditZoom` indicates the last zoom level used when editing the game in the engine's editor. See [Map Edit Zooms](#map-edit-zooms).

`FullPackageFlag` determines whether or not the runtime should attempt to load a runtime package before running the game. See [Runtime Package](#runtime-package).

## Map Edit Modes
These are the edit modes for every valid value of `MapEditMode`. Each mode corresponds to a layer within the editor.

| Value | Description            |
|:------|:-----------------------|
| `0`   | Lower tile layer mode. |
| `1`   | Upper tile layer mode. |
| `2`   | Event layer mode.      |

## Map Edit Zooms
These are the zoom levels for every valid value of `MapEditZoom`. Each mode corresponds to a zoom level (view ratio) within the editor.

The larger the number, the more zoomed out the editor will be.

| Value | Description     |
|:------|:----------------|
| `0`   | 1:1 view ratio. |
| `1`   | 1:2 view ratio. |
| `2`   | 1:4 view ratio. |
| `3`   | 1:8 view ratio. |

## Runtime Package
Runtime packages are installed on the player's machine and provide default assets for games to use;
games that make use of a runtime package will not work properly if the package is not installed on the user's machine or if they're not loaded by the runtime.
This is done to keep games from distributing the same set of assets, potentially reducing the amount of disk-space games take up.

If a game does not make use of a runtime package, then `FullPackageFlag` may be set to `1` to allow the runtime to skip loading
a runtime package, thereby improving startup performance and not requiring one to be installed. Otherwise, this field should be
set to `0` to instruct the runtime to attempt to load a runtime package.

Official runtime packages for RM2k/3 can obtained from the [official website](https://www.rpgmakerweb.com/download/additional/run-time-packages).

## Example Configuration File
The following is an example `RPG_RT.ini` file.

```text
[RPG_RT]
GameTitle=My Game
MapEditMode=2
MapEditZoom=0
FullPackageFlag=1
```

**Runtime:** This file will result in a runtime window titled "My Game" and the runtime will not attempt to load a runtime package.

**Editor:** When this file is loaded by the RM2k/3 editor, the edit mode will be set to the event layer (`2`), and the zoom mode will be 1:1 (`0`).
