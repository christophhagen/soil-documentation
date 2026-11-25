# Compact JSON Encoding

This document describes the compact JSON encoding format implemented in the SOIL core libraries.

## Structure

The compact JSON format is structurally the same as the [JSON Encoding](JSON.md), and follows the same rules.
It only differs in the field names, which are single characters for the compact format.
Additionally, the message and value types are encoded as single-digit integers, not as strings.

### Field names

The following table explains how the regular JSON field names are translated to the compact representation:

| JSON Field    | Compact name | Comment                                                           |
|:--------------|:-------------|:------------------------------------------------------------------|
| `children`    | `n`          | `n` is derived from `nodes`, since `c` is already used by `count` |
| `count`       | `c`          |                                                                   |
| `depth`       | `d`          |                                                                   |
| `end`         | `e`          |                                                                   |
| `id`          | `i`          |                                                                   |
| `message`     | `m`          |                                                                   |
| `path`        | `p`          |                                                                   |
| `samples`     | `s`          |                                                                   |
| `time`        | `t`          |                                                                   |
| `type`        | `h`          | `h` is derived from `hint`, since `t` is already used by `time`   |
| `value`       | `v`          |                                                                   |

The full field names of the regular JSON format are not recognized, and are ignored like any other unknown field.

### Message type

The message type is transmitted as an integer, according to the following table:

| JSON type | Compact value |
|:----------|:--------------|
| `data`    | `0`           |
| `read`    | `1`           |
| `delete`  | `2`           |
| `include` | `3`           |
| `exclude` | `4`           |
| `clear`   | `5`           |

### Value type

The value type is also transmitted as an integer:

| SOIL Type | JSON Type   | Compact value |
|:----------|:------------|:--------------|
| Null      | `"null"`    | `0`           |
| Integer   | `"integer"` | `1`           |
| Double    | `"double"`  | `2`           |
| String    | `"string"`  | `3`           |
| Data      | `"data"`    | `4`           |
| Path      | `"path"`    | `5`           |
| List      | `"list"`    | `6`           |
| Map       | `"map"`     | `7`           |

The codes are the same as the value types of the [binary encoding](Binary.md).

Message and value types are always integers, including the value types within [lists](JSON.md#list) and [maps](JSON.md#map).
Type names as strings are rejected.

Only the types are changed, the values themselves are encoded as in the regular JSON format.
In particular, non-finite doubles remain the strings `"nan"`, `"inf"` and `"-inf"` (with the type `2`),
and paths remain arrays of strings (or a single string).

Example: The list `[Int(123), Double(NaN), Path("a", "b")]` at the path `a` is represented as:

```json
{
    "p": ["a"],
    "h": 6,
    "v": [1, 123, 2, "nan", 5, ["a", "b"]]
}
```

## Value encoding

The [value encoding](JSON.md#value-encoding) also exists in a compact variant, which uses the integer type codes.
For example, a tagged integer is encoded as `[1,5]` instead of `["integer",5]`, and a tagged list as `[6,[1,1,3,"x"]]`.
The encoding of value content is identical to the regular JSON format.
