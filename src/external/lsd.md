# LCF Save Data File Specification
## Introduction
LCF Save Data (LSD) files are used to store save data for RPG Maker 2000/2003 (RM2k/3) games.
LSD files use a tag-based format similar to the one used in e.g. Adobe SWF files. This allows for easy reading and writing.

These files are typically stored within the same directory as the game's executable file (`RPG_RT.exe`).

## LCF Save Data File Structure
LMT files are stored in binary format.

The following table describes the overall structure of an LSD file.

| Field         | Type                                            | Description                                                |
|:--------------|:------------------------------------------------|:-----------------------------------------------------------|
| Signature     | `STRING`                                        | The file's signature; this should always be "LcfSaveData". |
| Timestamp     | `DATE`                                          | The total playtime.                                        |
|