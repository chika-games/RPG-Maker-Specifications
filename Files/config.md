# RPG Maker Config File Specification (RPG_RT.ini)
## Table of Contents
* Table of Contents
* [Introduction](#introduction)
* [Data Types](#data-types)
* [Config File Structure](#config-file-structure)
    * [Map Editing Modes](#map-editing-modes)
    * [Map Zoom Levels](#map-zoom-levels)
* [Example Config File](#example-config-file)
* [Legal Information](#legal-information)

## Introduction
The `RPG_RT.ini` configuation file provides basic start-up information for RPG Maker 2003 games. This file follows a simple [INI](https://en.wikipedia.org/wiki/INI_file) format.

This file is found within the same directory as the RPG Maker 2003 runtime (`RPG_RT.exe`). This file is not required, and, if not present, the runtime will use the default values listed below.

## Data Types
This section describes the various data types that will be used throughout this specification.

| Type | Description |
| --- | --- |
| STRING | A string of characters. |
| UINT | An unsigned integer. |
| BOOL | A boolean integer: `0` is false and `1` is true. |

## Config File Structure
This section details the overall structure of an `RPG_RT.ini` file.

The fields should be specified within an "RPG_RT" section. See the [Example Config File](#example-config-file) for a simple example.

None of the fields listed below are required and may be omitted; missing fields will take on a default value. Fields with invalid values will be ignored and the default value will be used.

These default values are based on the official RPG Maker runtime.

| Field | Type | Default Value | Description |
| --- | --- | --- | --- |
| GameTitle | STRING | "Untitled" | Determines the title of the game window. |
| MapEditMode | UINT | `0` | For editor use only. Indicates the last layer-editing mode used. This may help when re-opening a project; the last selected mode can automatically be re-selected. See [Map Editing Modes](#map-editing-modes). |
| MapEditZoom | UINT | `0` | For editor use only. Indicates the last zoom level used. This may help when re-opening a project; the last zoom level selected can automatically be re-selected. See [Map Zoom Levels](#map-zoom-levels). |
| FullPackageFlag | BOOL | `0` (false) | Determines whether or not to skip loading a runtime package (RTP); true (`1`) means do not load. |

### Map Editing Modes
| Mode | Value |
| --- | --- |
| Lower Layer | 0 |
| Upper Layer | 1 |
| Event Layer | 2 |

### Map Zoom Levels
| Level | Value |
| --- | --- |
| 1:1 Ratio | 0 |
| 1:2 Ratio | 1 |
| 1:4 Ratio | 2 |
| 1:8 Ratio | 3 |

## Example Config File
Below is an example configuration file for demonstration purposes.

```
[RPG_RT]
GameTitle=My Game
MapEditMode=2
MapEditZoom=0
FullPackageFlag=1
```

## Legal Information
[![Creative Commons License](https://i.creativecommons.org/l/by-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-sa/4.0/)

This document is licensed under the [CC BY-SA 4.0](http://creativecommons.org/licenses/by-sa/4.0/) license. Copyright (c) Chika Games.

The RPG Maker trademark and copyright are property of Enterbrain, Inc. and Kadokawa Corporation. All rights belong to their respective owners.
