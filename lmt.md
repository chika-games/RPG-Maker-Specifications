# LCF Map Tree Specification (LMT)
## Background

| Key | Value |
| --- | --- |
| Version | _UNDER CONSTRUCTION_ |
| Status | _Research/Speculation_ |
| License | [![Creative Commons License](https://i.creativecommons.org/l/by-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-sa/4.0/) |

## Overview

## Type Notation
This is a table of notations used within this document to denote various types of values:

| Notation | Description |
| --- | --- |
| STRING | An unsigned 8-bit `length` value followed by a contiguous chunk of `length`-many unsigned 8-bit integers. (I.e. a length followed by characters.) |
| EINT | An encoded variable-length integer. See [Encoded Integers](#encoded-integers) for more information. |

## Memory Layout
The following describes the overall structure of an LMT file from top to bottom:

| Name | Type | Description |
| --- | --- | --- |
| `Header` | STRING | The header always has a length of 10 and a value of `LcfMapTree`. This is used to determine if a file represents an LMT file. |
| `Info Count` | EINT | The number of [Map Information] structures to follow. |
| `Map Info` | [Map Information] [Info Count] | An array of [Map Information] structures. |
| `Order Count` | EINT | The number of map orderings to follow. |
| `Map Order` | EINT[`Order Count`] | An array of map IDs in sequential order. For example, the first map of a game is the first element in the array. |
| `Active Node` | EINT |  |
| `Start` | [Map Start](#map-start) | A structure containing information about starting maps and positions. |

Types may be appended with `[n]`, where n is a positive integer, in order to denote a contiguous array of said type. For example, `U8[4]` represents an array of four unsigned 8-bit integers.

[Map Information]: #map-information

### Encoded Integers
LCF files store integers in an encoded format. Below is pseudo-code for reading and decoding these integers:

```python
int read_encoded_integer(reader):
    var ret = 0
    
    while true:
        var b = reader.read_u8()
        
        ret <<= 7
        ret |= b & 0x7F
        
        if (b & 0x80) == 0:
            break
    
    return ret
```

(Based on code from [gabien-app-r48](https://github.com/20kdc/gabien-app-r48).)

### Map Information

### Map Start
