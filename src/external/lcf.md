# LCF Files
Many of the external files used by RM2k/3 follow a custom binary format called LCF.
This format uses a tag system for storing its contents, similar to Adobe SWF files.

Specifically, all LCF files contain some sort of header data (signature, date, etc.)
followed by a series of tagged data blocks. All tags - except the [End Tag](common_tags.md#end-tag) - share the same format.
This structure allows tags to be easily inserted, removed, and/or modified.

## Tag Format
All tags begin with a tag type and a tag size field followed by tag-specified data:

| Field   | Type   | Description                              |
|:--------|:-------|:-----------------------------------------|
| TagType | `EINT` | An ID that "uniquely" determines a tag.  |
| TagSize | `EINT` | The size in bytes of the data to follow. |

The `TagType` field "uniquely" determines the kind of tag being dealt with. This ID is only unique within a particular context;
for example, it may be the case that a tag dealing with music and a tag dealing with enemies share the same ID.
However, said tags will never be used within the same context/structure and thus can be distinguished from each other.

Following the above fields is zero or more fields depending on the tag. The `TagSize` contains the total size in bytes of the
fields to follow. This does not include the `TagType` field nor the `TagSize` field itself. This can be helpful when looking
for or modifying specific tags - unwanted tags can be skipped by simply reading then discarding `TagSize` number of bytes
after first reading the above two fields.

### Example Tag
As an example, consider the hypothetical tag that deals with playing a song:

| Field   | Type   | Description                              |
|:--------|:-------|:-----------------------------------------|
| TagType | `EINT` | An ID that "uniquely" determines a tag.  |
| TagSize | `EINT` | The size in bytes of the data to follow. |
| SongID  | `EINT` | The unique ID of the song to play.       |
| Volume  | `U8`   | Volume to play the song at.              |
| Tempo   | `U8`   | Tempo to play the song at.               |

The `TagType` field would have a "unique" ID, say `15`.
Since `Volume` and `Tempo` take up 2 bytes, the `TagSize` field would have a value of 2 + the size of `SongID` in bytes.

This tag could easily be skipped in the following way: first, read `TagType` and `TagSize`, then read `TagSize` number of bytes and discard them.
