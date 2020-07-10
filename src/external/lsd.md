# LCF Save Data File Specification
## Introduction
LCF Save Data (LSD) files are used to store save data for RM2k/3 games.

These files are always stored within the same directory as the RM2k/3 Runtime (i.e. `RPG_RT.exe`).
Several of these files may exist, each with the same naming scheme: "SaveXY.lsd" where XY is a number corresponding to the save number.

These files are only used in the RM2k/3 Runtimes.

## LCF Save Data File Structure
LSD files are stored in binary format.

The following table describes the overall structure of an LSD file.

| Field         | Type                                            | Description                                                |
|:--------------|:------------------------------------------------|:-----------------------------------------------------------|
| Signature     | `STRING`                                        | The file's signature; this should always be "LcfSaveData". |
| Timestamp     | `DATE`                                          | The total playtime.                                        |
