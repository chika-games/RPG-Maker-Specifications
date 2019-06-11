# LCF Map Tree Specification (LMT)
## Background

| Key | Value |
| --- | --- |
| Version | UNSTABLE |
| License | [![Creative Commons License](https://i.creativecommons.org/l/by-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-sa/4.0/) |

## Overview

## Type Notation
This is a table of notations used within this document to denote various types of values:

| Notation | Description |
| --- | --- |
| STRING | An unsigned 8-bit `length` value followed by a contiguous chunk of `length`-many unsigned 8-bit integers. (I.e. a length followed by characters.) |
| SPARSE |  |

## Memory Layout
The following describes the overall structure of an LMT file from top to bottom:

| Name | Type | Description |
| --- | --- | --- |
| `Header` | STRING | The header always has a length of 10 and a value of `LcfMapTree`. This is used to determine if a file represents an LMT file. |

## Attribution
