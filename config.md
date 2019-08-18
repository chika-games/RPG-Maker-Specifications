# RPG Maker Config File Specification (RPG_RT.ini)
| Key | Value |
| --- | --- |
| Version | _Researching_ |
| License | [CC BY-SA 4.0](http://creativecommons.org/licenses/by-sa/4.0/) |

## Table of Contents
* Table of Contents
* [Introduction](#introduction)
* [Data Types](#data-types)
* [Config File Structure](#config-file-structure)
* [Example Config File](#example-config-file)
* [Document Changes](#document-changes)
* [Legal Information](#legal-information)

## Introduction
The `RPG_RT.ini` configuation file provides basic startup information used by RPG Maker 2000 and 2003 when launching a game. This file follows a simple [INI](https://en.wikipedia.org/wiki/INI_file) format, as its file extension suggests.

## Data Types
This section describes the various data types that will be used throughout this specification.

| Type | Description |
| --- | --- |
| STRING | A string of characters. |
| INT | A signed integer. |
| BOOL | A boolean integer: `0` is false and `1` is true. |

## Config File Structure
This section details the overall structure of an `RPG_RT.ini` file.

The fields should be specified within an "RPG_RT" section. See the [Example Config File](#example-config-file) for a simple example.

None of the fields listed below are required and may be omitted; missing fields will take on a default value. These default values are based on the official RPG Maker runtime and are by no means standard.

| Field | Type | Default Value | Description |
| --- | --- | --- | --- |
| GameTitle | STRING | "Untitled" | Determines the title of the game window. |
| MapEditMode | INT | 0 | |
| MapEditZoom | INT | 0 | |
| FullPackageFlag | BOOL | false (`0`) | Determines whether or not to skip loading a runtime package (RTP); true (`1`) means do not load. |

## Example Config File
Below is an example configuration file for demonstration purposes.

```
[RPG_RT]
GameTitle=My Game
MapEditMode=2
MapEditZoom=0
FullPackageFlag=1
```

## Document Changes

## Legal Information
This document is provided under the [CC BY-SA 4.0](http://creativecommons.org/licenses/by-sa/4.0/) license.

The RPG Maker trademark and copyright are property of Enterbrain, Inc. and Kadokawa Corporation.

All rights belong to their respective owners.
