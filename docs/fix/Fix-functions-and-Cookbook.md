---
layout: default
title: Fix Functions and Cookbook
parent: Fix
nav_order: 2
---

This page is a replication of the passage of the Fix Readme.md.

## Functions and cookbook

### Best practices and guidelines for working with Metafacture Fix

- We recommend to use double quotation marks for arguments and values in functions, binds and conditionals.
- If using a `list` bind with a variable, the `var` option requires quotation marks (`do list(path: "<sourceField>", "var": "<variableName>")`).
- Fix turns repeated fields into arrays internally but only marked arrays (with `[]` at the end of the field name) are also emitted as "arrays" (entities with indexed literals), all other arrays are emitted as repeated fields.
- Every Fix file should end with a final newline.

### Glossary

#### Array wildcards

Array wildcards resemble [Catmandu's concept of wildcards](http://librecat.org/Catmandu/#wildcards).

When working with arrays and repeated fields you can use wildcards instead of an index number to select elements of an array.

| Wildcard | Meaning |
|----------|:--------|
| `*` | Selects _all_ elements of an array. |
| `$first` | Selects only the _first_ element of an array. |
| `$last` | Selects only the _last_ element of an array. |
| `$prepend` | Selects the position _before_ the first element of an array. Can only be used when adding new elements to an array. |
| `$append` | Selects the position _after_ the last element of an array. Can only be used when adding new elements to an array. |

#### Path wildcards

Path wildcards resemble [Metamorph's concept of wildcards](https://github.com/metafacture/metafacture-core/wiki/Metamorph-User-Guide#addressing-pieces-of-data). They are not supported in Catmandu (it has [specialized Fix functions](https://librecat.org/Catmandu/#marc-mab-pica-paths) instead).

You can use path wildcards to select fields matching a pattern. They only match path _segments_ (field names), though, not _whole_ paths of nested fields. These wildcards cannot be used to add new elements.

| Wildcard | Meaning |
|----------|:--------|
| `*` | Placeholder for zero or more characters. |
| `?` | Placeholder for exactly one character. |
| `\|` | Alternation of multiple patterns. |
| `[...]` | Enumeration of characters. |

### Functions

#### Script-level functions

##### `include`

Includes a Fix file and executes it as if its statements were written in place of the function call.

Parameters:

- `path` (required): Path to Fix file (if the path starts with a `.`, it is resolved relative to the including file's directory; otherwise, it is resolved relative to the current working directory).

Options:

- All options are made available as "dynamic" local variables in the included Fix file.

```perl
include("<path>"[, <dynamicLocalVariables>...])
```

##### `nothing`

Does nothing. It is used for benchmarking in Catmandu.

```perl
nothing()
```

##### `put_filemap`

Defines an external map for [lookup](#lookup) from a file or a URL. Maps with more than 2 columns are supported but are reduced to a defined key and a value column.

```perl
put_filemap("<sourceFile>", "<mapName>", sep_char: "\t")
```

The separator (`sep_char`) will vary depending on the source file, e.g.:

| Type | Separator  |
|------|------------|
| CSV  | `,` or `;` |
| TSV  | `\t`       |

Options:

- `allow_empty_values`: Sets whether to allow empty values in the filemap or to ignore these entries. (Default: `false`)
- `compression`: Sets the compression of the file.
- `decompress_concatenated`: Flags whether to use decompress concatenated file compression.
- `encoding`: Sets the encoding used to open the resource.
- `expected_columns`: Sets number of expected columns; lines with different number of columns are ignored. Set to `-1` to disable the check and allow arbitrary number of columns. (Default: `2`)
- `key_column`: Defines the column to be used for keys. Uses zero index. (Default: `0`)
- `value_column`: Defines the column to be used for values. Uses zero index. (Default: `1`)

##### `put_map`

Defines an internal map for [lookup](#lookup) from key/value pairs.

```perl
put_map("<mapName>",
  "dog": "mammal",
  "parrot": "bird",
  "shark": "fish"
)
```

##### `put_rdfmap`

Defines an external RDF map for lookup from a file or an HTTP(S) resource.
As the RDF map is reducing RDF triples to a key/value map it is mandatory to set the target.
The targeted RDF property can optionally be bound by an RDF language tag.

```perl
put_rdfmap("<rdfResource>", "<rdfMapName>", target: "<rdfProperty>")
put_rdfmap("<rdfResource>", "<rdfMapName>", target: "<rdfProperty>", select_language: "<rdfLanguageTag>")
```

##### `put_var`

Defines a single global variable that can be referenced with `$[<variableName>]`.

```perl
put_var("<variableName>", "<variableValue>")
```

##### `put_vars`

Defines multiple global variables that can be referenced with `$[<variableName>]`.

```perl
put_vars(
  "<variableName_1>": "<variableValue_1>",
  "<variableName_2>": "<variableValue_2>"
)
```

#### Record-level functions

##### `add_field`

Creates a field with a defined value.

```perl
add_field("<targetFieldName>", "<fieldValue>")
```

##### `array`

Converts a hash/object into an array.

```perl
array("<sourceField>")
```

E.g.:

```perl
array("foo")
# {"name":"value"} => ["name", "value"]
```

##### `call_macro`

Calls a named macro, i.e. a list of statements that have been previously defined with the [`do put_macro`](#do-put_macro) bind.

Parameters:

- `name` (required): Unique name of the macro.

Options:

- All options are made available as "dynamic" local variables in the macro.

```perl
do put_macro("<macroName>"[, <staticLocalVariables>...])
  ...
end
call_macro("<macroName>"[, <dynamicLocalVariables>...])
```

##### `copy_field`

Copies a field from an existing field.

```perl
copy_field("<sourceField>", "<targetField>")
```

##### `format`

Replaces the value with a formatted (`sprintf`-like) version.

---- TODO: THIS NEEDS MORE CONTENT -----

```perl
format("<sourceField>", "<formatString>")
```

##### `hash`

Converts an array into a hash/object.

```perl
hash("<sourceField>")
```

E.g.:
```perl
hash("foo")
# ["name", "value"] => {"name":"value"}
```

##### `move_field`

Moves a field from an existing field. Can be used to rename a field.

```perl
move_field("<sourceField>", "<targetField>")
```

##### `parse_text`

Parses a text into an array or hash of values.

---- TODO: THIS NEEDS MORE CONTENT -----

```perl
parse_text("<sourceField>", "<parsePattern>")
```

##### `paste`

Joins multiple field values into a new field. Can be combined with additional literal strings.

The default `join_char` is a single space. Literal strings have to start with `~`.

```perl
paste("<targetField>", "<sourceField_1>"[, ...][, "join_char": ", "])
```

E.g.:

```perl
# a: eeny
# b: meeny
# c: miny
# d: moe
paste("my.string", "~Hi", "a", "~how are you?")
# "my.string": "Hi eeny how are you?"
```

##### `print_record`

Prints the current record as JSON either to standard output or to a file.

Parameters:

- `prefix` (optional): Prefix to print before the record; may include [format directives](https://docs.oracle.com/javase/8/docs/api/java/util/Formatter.html#syntax) for counter and record ID (in that order). (Default: Empty string)

Options:

- `append`: Whether to open files in append mode if they exist. (Default: `false`)
- `compression` (file output only): Compression mode. (Default: `auto`)
- `destination`: Destination to write the record to; may include [format directives](https://docs.oracle.com/javase/8/docs/api/java/util/Formatter.html#syntax) for counter and record ID (in that order). (Default: `stdout`)
- `encoding` (file output only): Encoding used by the underlying writer. (Default: `UTF-8`)
- `footer`: Footer which is written at the end of the output. (Default: `\n`)
- `header`: Header which is written at the beginning of the output. (Default: Empty string)
- `id`: Field name which contains the record ID; if found, will be available for inclusion in `prefix` and `destination`. (Default: `_id`)
- `internal`: Whether to print the record's internal representation instead of JSON. (Default: `false`)
- `pretty`: Whether to use pretty printing. (Default: `false`)
- `separator`: Separator which is written after the record. (Default: `\n`)

```perl
print_record(["<prefix>"][, <options>...])
```

E.g.:

```perl
print_record("%d) Before transformation: ")
print_record(destination: "record-%2$s.json", id: "001", pretty: "true")
print_record(destination: "record-%03d.json.gz", header: "After transformation: ")
```

##### `random`

Creates (or replaces) a field with a random number (less than the specified maximum).

```perl
random("<targetField>", "<maximum>")
```

##### `remove_field`

Removes a field.

```perl
remove_field("<sourceField>")
```

##### `rename`

Replaces a regular expression pattern in subfield names of a field. Does not change the name of the source field itself.

```perl
rename("<sourceField>", "<regexp>", "<replacement>")
```

##### `retain`

Deletes all fields except the ones listed (incl. subfields).

```perl
retain("<sourceField_1>"[, ...])
```

##### `set_array`

Creates a new array (with optional values).

```perl
set_array("<targetFieldName>")
set_array("<targetFieldName>", "<value_1>"[, ...])
```

##### `set_field`

Creates (or replaces) a field with a defined value.

```perl
set_field("<targetFieldName>", "<fieldValue>")
```

##### `set_hash`

Creates a new hash (with optional values).

```perl
set_hash("<targetFieldName>")
set_hash("<targetFieldName>", "subfieldName": "<subfieldValue>"[, ...])
```

##### `timestamp`

Creates (or replaces) a field with the current timestamp.

Options:

- `format`: Date and time pattern as in [java.text.SimpleDateFormat](https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html). (Default: `timestamp`)
- `timezone`: Time zone as in [java.util.TimeZone](https://docs.oracle.com/javase/8/docs/api/java/util/TimeZone.html). (Default: `UTC`)
- `language`: Language tag as in [java.util.Locale](https://docs.oracle.com/javase/8/docs/api/java/util/Locale.html). (Default: The locale of the host system)

```perl
timestamp("<targetField>"[, format: "<formatPattern>"][, timezone: "<timezoneCode>"][, language: "<languageCode>"])
```

##### `vacuum`

Deletes empty fields, arrays and objects.

```perl
vacuum()
```

#### Field-level functions

##### `append`

Adds a string at the end of a field value.

```perl
append("<sourceField>", "<appendString>")
```

##### `capitalize`

Upcases the first character in a field value.

```perl
capitalize("<sourceField>")
```

##### `count`

Counts the number of elements in an array or a hash and replaces the field value with this number.

```perl
count("<sourceField>")
```

##### `downcase`

Downcases all characters in a field value.

```perl
downcase("<sourceField>")
```

##### `filter`

Only keeps field values that match the regular expression pattern. Works only with array of strings/repeated fields.

```perl
filter("<sourceField>", "<regexp>")
```

##### `flatten`

Flattens a nested array field.

```perl
flatten("<sourceField>")
```

##### `from_json`

Replaces the string with its JSON deserialization.

Options:

- `error_string`: Error message as a placeholder if the JSON couldn't be parsed. (Default: `null`)

```perl
from_json("<sourceField>"[, error_string: "<errorValue>"])
```

##### `index`

Returns the index position of a substring in a field and replaces the field value with this number.

```perl
index("<sourceField>", "<substring>")
```

##### `isbn`

Extracts an ISBN and replaces the field value with the normalized ISBN; optionally converts and/or validates the ISBN.

Options:

- `to`: ISBN format to convert to (either `ISBN10` or `ISBN13`). (Default: Only normalize ISBN)
- `verify_check_digit`: Whether the check digit should be verified. (Default: `false`)
- `error_string`: Error message as a placeholder if the ISBN couldn't be validated. (Default: `null`)

```perl
isbn("<sourceField>"[, to: "<isbnFormat>"][, verify_check_digit: "<boolean>"][, error_string: "<errorValue>"])
```

##### `join_field`

Joins an array of strings into a single string.

```perl
join_field("<sourceField>", "<separator>")
```

##### `lookup`

Looks up matching values in a map and replaces the field value with this match. [External files](#put_filemap), [internal maps](#put_map) as well as [RDF resources](#put_rdfmap) can be used.

Parameters:

- `path` (required): Field path to look up.
- `map` (optional): Name or path of the map in which to look up values.

Options:

- `__default`: Default value to use for unknown values. (Default: Old value)
- `delete`: Whether to delete unknown values. (Default: `false`)
- `print_unknown`: Whether to print unknown values. (Default: `false`)

Additional options when printing unknown values:

- `append`: Whether to open files in append mode if they exist. (Default: `true`)
- `compression` (file output only): Compression mode. (Default: `auto`)
- `destination`: Destination to write unknown values to; may include [format directives](https://docs.oracle.com/javase/8/docs/api/java/util/Formatter.html#syntax) for counter and record ID (in that order). (Default: `stdout`)
- `encoding` (file output only): Encoding used by the underlying writer. (Default: `UTF-8`)
- `footer`: Footer which is written at the end of the output. (Default: `\n`)
- `header`: Header which is written at the beginning of the output. (Default: Empty string)
- `id`: Field name which contains the record ID; if found, will be available for inclusion in `prefix` and `destination`. (Default: `_id`)
- `prefix`: Prefix to print before the unknown value; may include [format directives](https://docs.oracle.com/javase/8/docs/api/java/util/Formatter.html#syntax) for counter and record ID (in that order). (Default: Empty string)
- `separator`: Separator which is written after the unknown value. (Default: `\n`)

```perl
lookup("<sourceField>"[, <mapName>][, <options>...])
```

E.g.:

```perl
# local (unnamed) map
lookup("path.to.field", key_1: "value_1", ...)

# internal (named) map
put_map("internal-map", key_1: "value_1", ...)
lookup("path.to.field", "internal-map")

# external file map (implicit)
lookup("path.to.field", "path/to/file", sep_char: ";")

# external file map (explicit)
put_filemap("path/to/file", "file-map", sep_char: ";")
lookup("path.to.field", "file-map")

# RDF map (explicit)
put_rdfmap("path/to/file", "rdf-map", target: "<rdfProperty>")
lookup("path.to.field", "rdf-map")

# with default value
lookup("path.to.field", "map-name", __default: "NA")

# with printing unknown values to a file
lookup("path.to.field", "map-name", print_unknown: "true", destination: "unknown.txt")
```

##### `prepend`

Adds a string at the beginning of a field value.

```perl
prepend("<sourceField>", "<prependString>")
```

##### `replace_all`

Replaces a regular expression pattern in field values with a replacement string. Regexp capturing is possible; refer to capturing groups by number (`$<number>`) or name (`${<name>}`).

```perl
replace_all("<sourceField>", "<regexp>", "<replacement>")
```

##### `reverse`

Reverses the character order of a string or the element order of an array.

```perl
reverse("<sourceField>")
```

##### `sort_field`

Sorts strings in an array. Alphabetically and A-Z by default. Optional numerical and reverse sorting.

```perl
sort_field("<sourceField>")
sort_field("<sourceField>", reverse: "true")
sort_field("<sourceField>", numeric: "true")
```

##### `split_field`

Splits a string into an array and replaces the field value with this array.

```perl
split_field("<sourceField>", "<separator>")
```

##### `substring`

Replaces a string with its substring as defined by the start position (offset) and length.

```perl
substring("<sourceField>", "<startPosition>", "<length>")
```

##### `sum`

Sums numbers in an array and replaces the field value with this number.

```perl
sum("<sourceField>")
```

##### `to_json`

Replaces the value with its JSON serialization.

Options:

- `error_string`: Error message as a placeholder if the JSON couldn't be generated. (Default: `null`)
- `pretty`: Whether to use pretty printing. (Default: `false`)

```perl
to_json("<sourceField>"[, pretty: "<boolean>"][, error_string: "<errorValue>"])
```

##### `trim`

Deletes whitespace at the beginning and the end of a field value.

```perl
trim("<sourceField>")
```

##### `uniq`

Deletes duplicate values in an array.

```perl
uniq("<sourceField>")
```

##### `upcase`

Upcases all characters in a field value.

```perl
upcase("<sourceField>")
```

##### `uri_encode`

Encodes a field value as URI. Aka percent-encoding.

Options:

- `plus_for_space`: Sets whether "space" (` `) will be substituted by a "plus" (`+`) or be percent escaped (`%20`). (Default: `true`)
- `safe_chars`: Sets characters that won't be escaped. Safe characters are the ranges 0..9, a..z and A..Z. These are always safe and should not be specified. (Default: `.-*_`)

```perl
uri_encode("<sourceField>"[, <options>...])
```

E.g.:

```perl
uri_encode("path.to.field", plus_for_space:"false", safe_chars:"")
```

### Selectors

#### `reject`

Ignores records that match a condition.

```perl
if <condition>
  reject()
end
```

### Binds

#### `do list`

Iterates over each element of an array. In contrast to Catmandu, it can also iterate over a single object or string.

```perl
do list(path: "<sourceField>")
  ...
end
```

Only the current element is accessible in this case (as the root element).

When specifying a variable name for the current element, the record remains accessible as the root element and the current element is accessible through the variable name:

```perl
do list(path: "<sourceField>", "var": "<variableName>")
  ...
end
```

#### `do list_as`

Iterates over each _named_ element of an array (like [`do list`](#do-list) with a variable name). If multiple arrays are given, iterates over the _corresponding_ elements from each array (i.e., all elements with the same array index, skipping elements whose arrays have already been exhausted).

```perl
do list_as(element_1: "<sourceField_1>"[, ...])
  ...
end
```

E.g.:

```perl
# "ccm:university":["https://ror.org/0304hq317"]
# "ccm:university_DISPLAYNAME":["Gottfried Wilhelm Leibniz Universität Hannover"]
set_array("sourceOrga[]")
do list_as(orgId: "ccm:university[]", orgName: "ccm:university_DISPLAYNAME[]")
  copy_field(orgId, "sourceOrga[].$append.id")
  copy_field(orgName, "sourceOrga[].$last.name")
end
# {"sourceOrga":[{"id":"https://ror.org/0304hq317","name":"Gottfried Wilhelm Leibniz Universität Hannover"}]}
```

#### `do once`

Executes the statements only once (when the bind is first encountered), not repeatedly for each record.

```perl
do once()
  ...
end
```

In order to execute multiple blocks only once, tag them with unique identifiers:

```perl
do once("maps setup")
  ...
end
do once("vars setup")
  ...
end
```

#### `do put_macro`

Defines a named macro, i.e. a list of statements that can be executed later with the [`call_macro`](#call_macro) function.

Variables can be referenced with `$[<variableName>]`, in the following order of precedence:

1. "dynamic" local variables, passed as options to the `call_macro` function;
2. "static" local variables, passed as options to the `do put_macro` bind;
3. global variables, defined via [`put_var`](#put_var)/[`put_vars`](#put_vars).

Parameters:

- `name` (required): Unique name of the macro.

Options:

- All options are made available as "static" local variables in the macro.

```perl
do put_macro("<macroName>"[, <staticLocalVariables>...])
  ...
end
call_macro("<macroName>"[, <dynamicLocalVariables>...])
```

### Conditionals

Conditionals start with `if` in case of affirming the condition or `unless` rejecting the condition.

Conditionals require a final `end`.

Additional conditionals can be set with `elsif` and `else`.

```perl
if <condition(params, ...)>
  ...
end
```

```perl
unless <condition(params, ...)>
  ...
end
```

```perl
if <condition(params, ...)>
  ...
elsif
  ...
else
  ...
end
```

#### `contain`

##### `all_contain`

Executes the functions if/unless the field contains the value. If it is an array or a hash all field values must contain the string.

##### `any_contain`

Executes the functions if/unless the field contains the value. If it is an array or a hash one or more field values must contain the string.

##### `none_contain`

Executes the functions if/unless the field does not contain the value. If it is an array or a hash none of the field values may contain the string.

##### `str_contain`

Executes the functions if/unless the first string contains the second string.

#### `equal`

##### `all_equal`

Executes the functions if/unless the field value equals the string. If it is an array or a hash all field values must equal the string.

##### `any_equal`

Executes the functions if/unless the field value equals the string. If it is an array or a hash one or more field values must equal the string.

##### `none_equal`

Executes the functions if/unless the field value does not equal the string. If it is an array or a hash none of the field values may equal the string.

##### `str_equal`

Executes the functions if/unless the first string equals the second string.

#### `exists`

Executes the functions if/unless the field exists.

```perl
if exists("<sourceField>")
```

#### `in`

Executes the functions if/unless the field value [is contained in](https://perldoc.perl.org/perlop#Smartmatch-Operator) the value of the other field.

_Also aliased as [`is_contained_in`](#is_contained_in)._

#### `is_contained_in`

_Alias for [`in`](#in)._

#### `is_array`

Executes the functions if/unless the field value is an array.

#### `is_empty`

Executes the functions if/unless the field value is empty.

#### `is_false`

Executes the functions if/unless the field value equals `false` or `0`.

#### `is_hash`

_Alias for [`is_object`](#is_object)._

#### `is_number`

Executes the functions if/unless the field value is a number.

#### `is_object`

Executes the functions if/unless the field value is a hash (object).

_Also aliased as [`is_hash`](#is_hash)._

#### `is_string`

Executes the functions if/unless the field value is a string (and not a number).

#### `is_true`

Executes the functions if/unless the field value equals `true` or `1`.

#### `match`

##### `all_match`

Executes the functions if/unless the field value matches the regular expression pattern. If it is an array or a hash all field values must match the regular expression pattern.

##### `any_match`

Executes the functions if/unless the field value matches the regular expression pattern. If it is an array or a hash one or more field values must match the regular expression pattern.

##### `none_match`

Executes the functions if/unless the field value does not match the regular expression pattern. If it is an array or a hash none of the field values may match the regular expression pattern.

##### `str_match`

Executes the functions if/unless the string matches the regular expression pattern.

