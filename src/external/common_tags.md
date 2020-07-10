# Common Tags
Listed are various tags that are shared between the different Lcf file formats.

## End Tag
Marks the end of a structure, tag, or [LIST](types.md#list-type).

This tag only has an ID field. Because of this, when stored in a binary file,
this tag will merely be an single zero-valued byte.

| Field | Type   | Description            |
|:------|:-------|:-----------------------|
| TagID | `EINT` | This will always be 0. |
