# RPGMaker 2000/2003 Specifications
## Introduction
This repository contains specifications the various file formats used by RPG Maker 2000/2003, such as XYZ images and LMT map files.

### Table of Contents
* [Introduction](#introduction)
* [About The Specifications](#about-the-specifications)
* [RPG Maker Config File (RPG_RT.ini)](#rpg-maker-config-file-rpg_rtini)
* [XYZ Images (XYZ)](#xyz-images-xyz)
* [LCF Map Tree (LMT)](#lcf-map-tree-lmt)
* [LCF Save Data (LSD)](#lcf-save-data-lsd)
* [License](#license)

## About the Specifications
All of the specifications within this repository are written with independence in mind; all relevant information is included, and they make no reference to each other.

Each specification includes a table that specifies its __Version__ and __License__ information.

__Version__ provides the relevant document's version. This field has three potential values:
* _Researching_ indicates the specification is still being researched and drafted.
* _Finalizing_ means all relevant research has completed, and the specification is undergoing finalization.
* A completed specification will have a `major.minor.revision` version format.

__License__ provides a specification's copyright license. All specifications are provided under the [CC BY-SA 4.0](http://creativecommons.org/licenses/by-sa/4.0/) license unless explicitly stated otherwise.

## RPG Maker Config File (RPG_RT.ini)
The `RPG_RT.ini` file provides basic game information, such as the title.

[__Specification__](config.md)

## XYZ Images (XYZ)
XYZ files are used by RPG Maker 2000/2003 to store image data.

[__Specification__](xyz.md)

## LCF Map Tree (LMT)
LMT files are used to specify various map properties, map orderings, and game start information.

[__Specification__](lmt.md)

## LCF Save Data (LSD)
LSD files are used to store save data.

[__Specification__](lsd.md) (_Researching_)

## License
[![Creative Commons License](https://i.creativecommons.org/l/by-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-sa/4.0/)

This document is licensed under the [CC BY-SA 4.0](http://creativecommons.org/licenses/by-sa/4.0/) license.

The RPG Maker trademark and copyright are property of Enterbrain, Inc. and Kadokawa Corporation. All rights belong to their respective owners.
