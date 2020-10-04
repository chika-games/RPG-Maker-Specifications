# Common Tags
Below are various tags that are shared between the different LCF file formats.

For more information on tags, please consult the [LCF page](lcf.md).

## End Tag
Marks the end of a structure, tag, or [LIST](data_types.md#list-type).

This tag only has an ID field. Because of this, when stored in a binary file,
this tag will be stored as a `U8` with a value of `0`.

| Field | Type   | Description            |
|:------|:-------|:-----------------------|
| TagID | `EINT` | This will always be 0. |
