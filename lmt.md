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
| EINT | An encoded integer value of variable length. See [Encoded Integer](#encoded-integer) for more information.
| SPARSE |  |

## Memory Layout
The following describes the overall structure of an LMT file from top to bottom:

| Name | Type | Description |
| --- | --- | --- |
| `Header` | STRING | The header always has a length of 10 and a value of `LcfMapTree`. This is used to determine if a file represents an LMT file. |

### Encoded Integer
LCF files store integers in an encoded format. Below is a pseudocode function for reading and decoding these integers:

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
