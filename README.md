# uSX 1.0 Specification

**DO NOT USE THIS YET, IT'S A DRAFT**

Last updated: 2017-02-07 14:27:28 CET

<!--
-ECF extensible configuration format
-UCF micro configuration format
-MCF minimal configuration format
CCF comprehensive configuration format
DMCF dead minimal configuration format
    ~DMCA digital millenium copyright act
DSSF dead simple structure format
DSS dead simple structures
DMS dead minimal structure
DSR dead simple records
+DMR dead minimal records
+DES dead extensible structures
+DXS dead extensible structures
USE micro structure extensible
+USX micro structure extensible
MSE micro structure extensible
MSX micro structure extensible

EDF extensible declarative format
UDF micro declarative format
MDF minimal declarative format
CDF comprehensive declarative format
-->

Micro Structure eXtensible (uSX) is a well-defined universal markup-like format for any data block (e.g. a configuration file) or stream (e.g. communication through a socket). Think of it as a counterpart to CommonMark, XML, Java properties, INI, etc. It's suitable for both embedded (including IoT) as well as huge setups (including high performance and databases). It's also perfectly suitable for efficient stream (sequential) processing (there are no global closing tags nor anything similar).

Its goals are easy readability by humans as well as parsability by computers and support for both character-based encodings and binary encodings. It might be also used as a serialization (marshalling) format (efficiency can be as high as with top serialization formats as uSX can easily embed them - see enhancement proposals below). That all while maintaining other qualities like being KISS, quick processing and creation, infinite streams support, interoperability with other formats, extensibility, compatibility with text editors and programming languages, etc.

The uSX **data model** is an unbounded list of records, where each record is a pair of an ID (a key) and a value. Because it's a list, the order of the pairs matters and a key can be used more than once in the list (each time with an arbitrary value). Non-generic (i.e. application-specific) uSX implementations might choose to not allow reuse of keys.

In this specification, a *character*, an *octet* and an *8bit fixed-sized value* have the same meaning. Valid uSX data must be parsable by an **octet-based** parser (other than octet-based encodings of characters **must conform** to this requirement).

The format uses just three characters (`'` `^` `LF`) as delimiters. Any character might be used as value (there are no requirements on encoding nor anything else) provided the following conditions are met.

The data are seen as a sequence of records. The record might be a one-line record or a multiline record. Each record is identified by an ID, which is a string starting at a new line (optionally indented by `\t` (tabelator) or ` ` (space) or their arbitrary combination) with `.` (dot) followed by a string matching the ERE `[_A-Za-z][_A-Za-z0-9]*([.][_A-Za-z][_A-Za-z0-9]*)*`.

For a one-line record, `'` follows, for a multiline record, `^` followed by a *string of maximum of 64 octets* and `LF` follows. For a one-line record any data follow until the first occurence of `LF`. For a multiline record any data follow until the first occurence of `LF` immediately followed by the *string of maximum of 64 octets*. Multiline records can be concatenated with any one-line or multiline record if `'` or `^` respectively are used immediately after the *string of maximum of 64 octets*. Comments are records without identification and thus do not start with `.`, but directly with `'` (one-line) or `^` (multiline).

- [BOM](https://en.wikipedia.org/wiki/Byte_order_mark ) is not allowed.
- The first line of the stream or file must be a one-line comment beginning with the string `<major>.<minor>` specifying the minimum required version of uSX to correctly interpret the data. Note, uSX does not offer any mechanism for a live transition to a newer version of uSX.
- Dots are used as visual separators, but have no meaning and are thus fully optional except for the very first dot on a line.
- `CRLF` is treated as two separate characters and thus only `LF` matters.
- The prefix matching ERE `^[ \t]*` before an ID or before a comment or before `LF` (i.e. on a blank line) is allowed.
- The prefix `.std` of IDs is reserved for future use.
- Implementations shall preserve comments in the AST after parsing (e.g. in case the AST is a tree, comments can be added as children to the last parsed non-comment item or as children of the top-most node or as a preceding sibling of the following node).
- The prefered file extension is *.usx* (FIXME register it at IANA).
- It's highly recommended to add a human-readable description of the format of the value of each record (preferably using a formal language) to a comment preceding the record.
- Users should consider using a short prefix for the set of records they use (the set might be the name of the platform, the name of the application, ...).

Example valid uSX data:
~~~~
'1.0

.abc.def.var00'a boring one liner
.variables.must.start.with.a.dot.at.the.beginning'another boring one liner

' one-line comment after arbitrary number of empty lines
.dots.are.optional.and.have.no.meaning.but.are.recommended.to.visuall.separate.the.key.into.logical.blocks^LIMITED_LENGTH_STRING_TO_64_CHARACTERS
some example
multiline
content
LIMITED_LENGTH_STRING_TO_64_CHARACTERS

^X
multiline
comment with
some useless line
X

       .this.id.starts.after.several.spaces^END
content
END^ABC
concatenated content containing
END
in the middle
ABC^DEF
yet another content containing
ABC
in the middle
DEF

.pleas.keep.ids.as.short.as.possible^ABC
ABC'concatenated content until LF

.abc.def.var04'last content
~~~~

# Enhancements for versions higher than 1.0

Note, uSX 1.0 does not (and will not) have any type information.

All enhancement proposals shall be backwards compatible. If you find a use case, which can't be handled easily enough with the current uSX version, then it's a fatal failure of uSX and a new version (potentially incompatible) shall be designed.

## Standard prefixes for common data structures

The prefix `.std` might be used only in the following forms. The use of a particular form forces implementations to obey the type information. If the format specification is versioned, then the name should contain as much from the version information as is needed for backwards compatibility (e.g. in case of [Semver](http://semver.org/ ) versioning `My Format 1.2.3` becomes either `.std.myformat1.2` or a lengthy variant `.std.my.format.1.2`). Each implementation must support at least the `.std.msgpack` prefixes.

### .std.usx

There are two variants of `.std.usx` types: the character-based `.std.usx.c` and the binary `.std.usx.b`. The difference is, that the character-based one must represent the value in a human-readable format (usually sacrificing performance and size). See comment of each type to see how are they encoded in a character-based form and a binary one (including endianness).

#### Scalar types

For character-based types, only the "biggest" or most comprehensive ones make sense (e.g. no fixed length ints), but for the binary ones it's vice versa.

* FIXME description
* `.std.usx.c.octetstream.<id>`
* FIXME isn't character and binary variant the same in this case?
* `.std.usx.b.octetstream.<id>`
* FIXME define encoding
* `.std.usx.c.bigint.<id>`
* `.std.usx.b.bigint.<id>`
* FIXME define encoding
* `.std.usx.c.bigfloat.<id>`
* `.std.usx.b.bigfloat.<id>`

#### Compound types

FIXME how should a universal syntax for any compound type look like? To get an overview what's expected, then take a look e.g. at all [Dao built-in types](http://daoscript.org/help/en/dao.type.html ). Don't forget about IDs.

* Homogeneous array of either compound types or scalars
* `.std.usx.c.array.`
* Homogeneous map with ID being the key
* `.std.usx.c.map.`
* Homogeneous Map with any compound or scalar type being the key and any compound or scalar type being the value
* `.std.usx.c.anymap.key.`
* `.std.usx.c.anymap.val.`
* Homogeneous set
* `.std.usx.c.set.`
* Tag name must conform to the uSX ID specification
* Tags do not have any value.
* Tags are applied to the first non-tag record after the tag specification.
* Binary variant is the same (because the value is anyway empty).
* `.std.usx.c.tag.<tag_part0>.<tag_part1>.<...>`
* `.std.usx.b.tag.<tag_part0>.<tag_part1>.<...>`

### .std.msgpack1

[MessagePack](http://msgpack.org/ )

#### Scalar types

* FIXME description
* `.std.msgpack`

#### Compound types

* FIXME description
* `.std.msgpack`

### .std.capnproto

[Cap'n Proto](https://capnproto.org/ )

#### Scalar types

* FIXME description
* `.std.capnproto`

#### Compound types

* FIXME description
* `.std.capnproto`

### .std.protobuf

[Protocol Buffers](https://github.com/google/protobuf )

#### Scalar types

* FIXME description
* `.std.protobuf`

#### Compound types

* FIXME description
* `.std.protobuf`

## Standard for efficient compression

Maybe make a trie from all record IDs and save it only once as the first record (`.std.meta.trie`) and then use shortest possible binary references instead of full human-readable ID names. Should we allow more characters in IDs? If so, then which ones?

## Introduce standards for 1:1 bidirectional mappings from/to most common serialization/configuration/... formats

Maybe use for these mappings a prefix `.std.<format>.<...>` (e.g. `.std.xml.node.tag` or `.std.xml.node.document`) in case we would want to guarantee correct interpretation by the reader.

1. XML

   Tag names and attribute names need to be saved as values. There is also another issue with tag attributes - how should we map their name and value while preserving the nesting? Probably like another record on a new line with the same nesting level as the tag. But how about attribute names (they can contain more characters than uSX IDs)? We might make it complicated and introduce each record for each atomical part of XML (this will be inefficient because of many long prefixes as each record will have one).

# Authors

1. **Jan Pacner**, Idego Global Ltd.
