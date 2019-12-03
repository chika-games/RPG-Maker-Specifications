# RPG Maker 2003 Specifications
## Introduction
This repository aims to completely document RPG Maker 2003, from its editor and runtime to its various files formats, such as XYZ images and LMT map files.

### Table of Contents
* [Introduction](#introduction)
* [About The Specifications](#about-the-specifications)
* [File Specifications](#file-specifications)
  * [RPG Maker Config File (RPG_RT.ini)](#rpg-maker-config-file-rpg_rtini)
  * [XYZ Images (XYZ)](#xyz-images-xyz)
  * [LCF Map Tree (LMT)](#lcf-map-tree-lmt)
  * [LCF Save Data (LSD)](#lcf-save-data-lsd)
* [License](#license)

## About the Specifications
All specifications within this repository are written with independence in mind and cover a specific aspect of RPG Maker 2003m All relevant information should be included, and they make no reference to each other.

Specifications within the `Editor Specs` directory deal with the RPG Maker 2003 editor.

Specifications within the `Runtime Specs` directory deal with the RPG Maker 2003 runtime (`RPG_RT`).

Specifications within the `File Specs` directory deal with the various custom file formats used by RPG Maker 2003.

## File Specifications

### RPG Maker Config File (RPG_RT.ini)
The `RPG_RT.ini` file provides basic game information, such as the game's title.

[__Specification__](File Specs/config.md)

### XYZ Images (XYZ)
XYZ files are used by RPG Maker 2003 to store image data.

[__Specification__](File Specs/xyz.md)

### LCF Map Tree (LMT)
LMT files are used to specify various map properties, map orderings, and game start information.

[__Specification__](File Specs/lmt.md)

### LCF Save Data (LSD)
LSD files are used to store save data.

[__Specification__](File Specs/lsd.md) (_Researching_)

## License
[![Creative Commons License](https://i.creativecommons.org/l/by-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-sa/4.0/)

This document is licensed under the [CC BY-SA 4.0](http://creativecommons.org/licenses/by-sa/4.0/) license.

The RPG Maker trademark and copyright are property of Enterbrain, Inc. and Kadokawa Corporation. All rights belong to their respective owners.
