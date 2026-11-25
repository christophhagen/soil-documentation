# JSON Encoding

This document describes the JSON encoding format implemented in the SOIL core libraries.

### Table of contents

- [Message types](#message-types)
  - [Data messages](#data-messages)
  - [Request messages](#request-messages)
  - [Deletion requests](#data-deletion-messages)
  - [Subscription inclusion](#subscription-inclusion-messages)
  - [Subscription exclusion](#subscription-exclusion-messages)
  - [Subscription deletion](#subscription-deletion-messages)
- [Message fields](#message-fields)
  - [Message type](#message-type)
  - [Path](#path)
  - [Value](#value)
  - [Value type](#value-type)
  - [Time](#time)
  - [Samples](#samples)
  - [Count](#count)
  - [Depth](#depth)
  - [End](#end)
  - [Id](#id)
  - [Children](#children)
- [Decoding rules](#decoding-rules)
- [Examples](#examples)
- [Value encoding](#value-encoding)

## Message types

There are currently six different messages in SOIL:

| Message Type                                               | Usage                                       |
|:-----------------------------------------------------------|:--------------------------------------------|
| [Data message](#data-messages)                             | Transmit a tree for a path                  |
| [Request message](#request-messages)                       | Request a tree, including history           |
| [Deletion message](#data-deletion-messages)                | Delete parts of a tree, including history   |
| [Subscription inclusion](#subscription-inclusion-messages) | Request future updates for a path or tree   |
| [Subscription exclusion](#subscription-exclusion-messages) | Prevent future updates for a path or tree   |
| [Subscription deletion](#subscription-deletion-messages)   | Clear subscription rules for a path or tree |

### Data messages

Data messages are used to transmit data.
The following fields are currently used:

| Field                      | Description                                                                 |
|:---------------------------|:----------------------------------------------------------------------------|
| [`message`](#message-type) | Omitted, or `data`                                                          |
| [`path`](#path)            | The top-level path of the transmitted data                                  |
| [`value`](#value)          | A value of the top-level `path`                                             |
| [`type`](#value-type)      | The data type of `value`                                                    |
| [`time`](#time)            | The timestamp of the `value`, or of the message if `value` is absent        |
| [`samples`](#samples)      | Samples associated with `path`                                              |
| [`count`](#count)          | Total number of grouped messages                                            |
| [`id`](#id)                | A unique id to identify the message, for responses or to group messages     |
| [`children`](#children)    | Additional data beneath the top level path. Used to construct complex trees |

### Request messages

Request messages are sent to SOIL components to request data.
The following fields are currently used:

| Field                      | Description                                                    |
|:---------------------------|:---------------------------------------------------------------|
| [`message`](#message-type) | Must be `read`                                                 |
| [`path`](#path)            | The requested top-level path for which data is requested       |
| [`time`](#time)            | The start time of the requested interval (exclusive)           |
| [`count`](#count)          | The maximum number of requested samples per node               |
| [`depth`](#depth)          | The number of levels below the `path` for which to return data |
| [`end`](#end)              | The end of the requested time interval (inclusive)             |
| [`id`](#id)                | A unique id to identify the request, used in response message  |

### Data deletion messages

These messages are used to request the deletion of data from supported nodes.
The following fields are currently used:

| Field                      | Description                                                  |
|:---------------------------|:-------------------------------------------------------------|
| [`message`](#message-type) | Must be `delete`                                             |
| [`path`](#path)            | The top-level path for which data is deleted                 |
| [`time`](#time)            | The start time of the interval to delete (exclusive)         |
| [`depth`](#depth)          | The number of levels below `path` to include in the deletion |
| [`end`](#end)              | The end of the time interval to delete (inclusive)           |

### Subscription inclusion messages

These messages are used to subscribe to future updates.
The following fields are currently used:

| Field                      | Description                                                      |
|:---------------------------|:-----------------------------------------------------------------|
| [`message`](#message-type) | Must be `include`                                                |
| [`path`](#path)            | The top-level path to subscribe to                               |
| [`depth`](#depth)          | The number of levels below `path` to include in the subscription |

### Subscription exclusion messages

These messages are used to unsubscribe from future updates.
The following fields are currently used:

| Field                      | Description                                                        |
|:---------------------------|:-------------------------------------------------------------------|
| [`message`](#message-type) | Must be `exclude`                                                  |
| [`path`](#path)            | The top-level path to unsubscribe from                             |
| [`depth`](#depth)          | The number of levels below `path` to exclude from the subscription |

### Subscription deletion messages

These messages are used to delete subscription rules.
The following fields are currently used:

| Field                      | Description                                                         |
|:---------------------------|:--------------------------------------------------------------------|
| [`message`](#message-type) | Must be `clear`                                                     |
| [`path`](#path)            | The top-level path for which to remove the subscription rules       |
| [`depth`](#depth)          | The number of levels below `path` for which to also clear the rules |

The other message fields (`time`, `count`, `depth`, `end` and `id`) can be used with every message type, even if they currently have no meaning for that type, e.g. an `id` in a deletion message.
Encoders write every field that is set, and decoders read every field that is present, regardless of the message type.
This way, encoders and decoders don't need to change if the fields are used by more message types in the future.
Components ignore fields that they don't use.

The data fields (`value`, `type`, `samples` and `children`) are only used in data messages.
They are ignored in all other messages, and should not be sent.

## Message Fields

All six messages are realized using a single JSON object, with the following top-level fields:

| Field                      | Description                                                                       |
|:---------------------------|:----------------------------------------------------------------------------------|
| [`message`](#message-type) | Specifies the type of message                                                     |
| [`path`](#path)            | The top-level path                                                                |
| [`value`](#value)          | A value being transmitted                                                         |
| [`type`](#value-type)      | The data type of `value`                                                          |
| [`time`](#time)            | The timestamp of the `value` or message, or the start of a time interval          |
| [`samples`](#samples)      | Samples associated with the `path`                                                |
| [`count`](#count)          | Total number of grouped messages, or maximum number of requested samples per node |
| [`depth`](#depth)          | Specifies the depth of a data request, deletion, or subscription                  |
| [`end`](#end)              | The end of a time interval for requests or deletions                              |
| [`id`](#id)                | A unique id to identify the message, for responses or to group messages           |
| [`children`](#children)    | Additional data beneath the top level path. Used to construct complex trees       |

### Message type

| Name          | `"message"` |
|:--------------|:------------|
| Type          | Text        |
| Optional      | Yes         |
| Default value | `"data"`    |

The `message` field identifies the type of the message, e.g. if the message represents a request for data, or transmits a tree.
The six different types of messages are represented by strings according to the following table:

| String    | Message Type                                               |
|:----------|:-----------------------------------------------------------|
| `data`    | [Data message](#data-messages)                             |
| `read`    | [Request message](#request-messages)                       |
| `delete`  | [Deletion message](#data-deletion-messages)                |
| `include` | [Subscription inclusion](#subscription-inclusion-messages) |
| `exclude` | [Subscription exclusion](#subscription-exclusion-messages) |
| `clear`   | [Subscription deletion](#subscription-deletion-messages)   |

This field is optional. If the field is not contained in a message, then the message is treated as a data message (`data`).
Unknown message types are rejected.

Example:

```json
{
    "message": "data"
}
```

### Path

| Name          | `"path"`                |
|:--------------|:------------------------|
| Type          | `Array<Text>` or `Text` |
| Optional      | Yes                     |
| Default value | `[]`                    |

The `path` field is used for all six message types:

| Type                                                       | Usage                                                                        |
|:-----------------------------------------------------------|:-----------------------------------------------------------------------------|
| [Data message](#data-messages)                             | The path to the root node of the tree in the message                         |
| [Request message](#request-messages)                       | The path to the root node of the requested tree                              |
| [Deletion message](#data-deletion-messages)                | The path to the root node of the data to delete                              |
| [Subscription inclusion](#subscription-inclusion-messages) | The path to the root node of the tree to subscribe to                        |
| [Subscription exclusion](#subscription-exclusion-messages) | The path to the root node of the tree to unsubscribe from                    |
| [Subscription deletion](#subscription-deletion-messages)   | The path to the root node of the tree for which to delete subscription rules |

The field is named `"path"` and expects a JSON array of text values, or a single string.
This field is optional. If the field is not present in a message, then an empty path is assumed.

Example:

```json
{
    "path": ["path", "of", "data"]
}
```

Single-component paths can also be transmitted as a plain string.
The following messages are equivalent:

```json
{
    "path": ["single"]
}
```

```json
{
    "path": "single"
}
```

### Value

| Name          | `"value"`                      |
|:--------------|:-------------------------------|
| Type          | Text/Number/Boolean/Null/Array |
| Optional      | Yes                            |
| Default value | `null`, if `type` is `"null"`  |

The `value` field is only used for [data messages](#data-messages).
It contains a value associated with the path specified in the [`path`](#path) field.
The SOIL type of the contained value may be further specified with the [`type`](#value-type) field.
JSON types are mapped to SOIL types in the following way:

| SOIL Type | JSON Type                | [`type`](#value-type)  | Comment                                                                                 |
|:----------|:-------------------------|:-----------------------|:----------------------------------------------------------------------------------------|
| Null      | `null`                   | omitted, or `"null"`   |                                                                                         |
| Integer   | `Number` or `Boolean`    | `"integer"`            | Booleans are decoded as `1` (`true`) and `0` (`false`), see below                       |
| Double    | `Number` or `Text`       | omitted, or `"double"` | Only non-finite numbers are represented as text: `"nan"`, `"-inf"`, and `"inf"`. These require the type. |
| String    | `Text`                   | omitted, or `"string"` |                                                                                         |
| Data      | `Text`                   | `"data"`               | String must contain base64-encoded data, see [Data](#data)                              |
| Path      | `Array<Text>` or `Text`  | omitted, or `"path"`   | Text can be used for single component paths, but then requires the type `"path"`       |
| List      | `Array`                  | `"list"`               | See [List](#list)                                                                       |
| Map       | `Array`                  | `"map"`                | See [Map](#map)                                                                         |

Without a `type`, a JSON number is decoded as a Double, a string as a String, and an array as a Path.

SOIL represents booleans as integers.
JSON booleans (`true` and `false`) are therefore decoded as the integers `1` and `0`,
both without type and with the type `"integer"`.
Booleans are rejected for all other types.
Encoders always write integers, never booleans.

JSON objects are not valid values and are rejected.

The value field can be used if only a single value/sample is transmitted for the path specified in the message.
It is permitted to transmit additional values using the [`samples`](#samples) field, or to only use the `samples` field.
An optional timestamp associated with the value is transmitted using the [`time`](#time) field.

A data message (or [child node](#children)) contains a sample at its top level if the field `value` is present, or if `type` is `"null"`.
If `value` is missing, then the sample has a `null` value.
A `type` other than `"null"` without a `value` is rejected.
A [`time`](#time) without `value` and `type` is the timestamp of the message (see [time](#time)), and is ignored in child nodes.

Example:

```json
{
    "value": 123.4
}
```

#### Data

Binary data is encoded as base64 text using the standard alphabet (`A`-`Z`, `a`-`z`, `0`-`9`, `+` and `/`, see [RFC 4648, Section 4](https://www.rfc-editor.org/rfc/rfc4648#section-4)).
The text must be padded with `=` to a multiple of four characters, and must not contain line breaks or other whitespace.
The URL-safe alphabet (`-` and `_`) is not supported.

Example: The bytes `0x01 0x02 0x03` are encoded as `"AQID"`.

#### List

Lists of values are encoded as a flat array of alternating types and values.
The type is the string associated with the stored value (see [Value Type](#value-type)),
while the value is the encoded value (see [Value](#value)).

Example:

The list `[Int(123), Path("a", "b")]` is represented as:

```json
{
    "type": "list",
    "value": [
        "integer", 123,
        "path", ["a", "b"]
    ]
}
```

Note that `null` values are represented as `"null", null`.

#### Map

Maps are encoded as a flat array of (key, type, value) triplets.
The key is a string, `type` is the [value type string](#value-type), and `value` is the encoded [Value](#value).

The map `{ "abc" -> Int(123), "def" -> String("something") }` is represented as:

```json
{
    "type": "map",
    "value": [
        "abc", "integer", 123,
        "def", "string", "something"
    ]
}
```

Note that `null` values are represented as `"null", null`.

The entries can appear in any order.
If a key appears multiple times, then the last entry is used.

### Value Type

| Name          | `"type"` |
|:--------------|:---------|
| Type          | Text     |
| Optional      | Yes      |
| Default value | N/A      |

The `type` field is used only in connection with the `value` field, and identifies the type of the `value` field.
The value of the field is always a string; a JSON `null` is not a valid type.

| SOIL Type | `type`      | Required                                            |
|:----------|:------------|:----------------------------------------------------|
| Null      | `"null"`    | no                                                  |
| Integer   | `"integer"` | yes                                                 |
| Double    | `"double"`  | only for non-finite values, which are encoded as text |
| String    | `"string"`  | no                                                  |
| Data      | `"data"`    | yes                                                 |
| Path      | `"path"`    | only for single component paths encoded as text     |
| List      | `"list"`    | yes                                                 |
| Map       | `"map"`     | yes                                                 |

The `type` can only appear without a `value` field if it is `"null"`.
This represents a sample with a `null` value, the same as `"value": null`.
Unknown types are rejected.

The value type is also used in [`List`](#list) and [`Map`](#map) representations, where it is always required.

Example:

```json
{
    "type": "integer"
}
```

### Time

| Name          | `"time"`      |
|:--------------|:--------------|
| Type          | Number        |
| Optional      | Yes           |
| Default value | `null`        |

The `time` field contains an optional timestamp, encoded as a floating-point number of the seconds since 1.1.1970 (UTC).
Non-finite timestamps are not allowed.
It has different purposes depending on the message type:

| Type                                        | Usage                                                     | Interpretation of missing value |
|:--------------------------------------------|:----------------------------------------------------------|:--------------------------------|
| [Data message](#data-messages)              | The timestamp associated with the [`value`](#value) field, or the timestamp of the message, if no `value` is present | Current time |
| [Request message](#request-messages)        | The start time of the requested interval (exclusive)      | Beginning of time               |
| [Deletion message](#data-deletion-messages) | The start time of the interval to delete (exclusive)      | Beginning of time               |

The `time` field is optional.
If it is not present, then the timestamp is decoded as `null`, which is interpreted as indicated in the table above.
Decoders do not replace a missing timestamp with the current time.

In data messages and [child nodes](#children), the `time` only applies to the `value` at the same level.
It is not applied to the entries in [`samples`](#samples).

If a data message contains a `time` without a `value` (or `"type": "null"`), then it is the timestamp of the message itself,
e.g. to indicate when the message was created, or when a request was processed.
To transmit both a message timestamp and samples, the samples are placed in the [`samples`](#samples) field:

```json
{
    "path": ["a"],
    "time": 123456789.012,
    "samples": [
        {
            "value": 1,
            "type": "integer",
            "time": 123456788.0
        }
    ]
}
```

In [child nodes](#children), a `time` without a `value` (or `"type": "null"`) is ignored.

Example:

```json
{
    "time": 123456789.012
}
```

### Samples

| Name          | `"samples"`   |
|:--------------|:--------------|
| Type          | Array         |
| Optional      | Yes           |
| Default value | `[]`          |

The `samples` field only applies to [data messages](#data-messages),
and can be used to transmit multiple samples (value + timestamp) for the path specified with the [`path`](#path) field.
The expected JSON structure for the field is an array of objects.
Each object represents a sample, with the following fields:

| Field   | Description                                                                           |
|:--------|:--------------------------------------------------------------------------------------|
| `value` | The value associated with the sample, see [value field](#value)                       |
| `type`  | The type of the value, see [value type](#value-type) (Note the different field name!) |
| `time`  | The timestamp associated with the value, see [time](#time)                            |

Each sample must contain a `value`, or the `type` `"null"` (see [value](#value)).
Objects with neither are ignored, even if they contain a `time`.
A missing `time` is decoded as `null` (the current time).
Other fields in sample objects are ignored.

Samples use the same fields as the top-level object (`value`, `type`, `time`).
The following messages are equivalent:

```json
{
    "path": ["my", "value"],
    "value": 123,
    "type": "integer",
    "time": 123456789.012
}
```

```json
{
    "path": ["my", "value"],
    "samples": [
        {
            "value": 123,
            "type": "integer",
            "time": 123456789.012
        }
    ]
}
```

The former definition is available to make messages simpler in cases where only a single value is to be transmitted.
If both forms are used, then the samples are combined.

### Count

| Name          | `"count"` |
|:--------------|:----------|
| Type          | Number    |
| Optional      | Yes       |
| Default value | Null      |

The `count` field is applicable to the following message types:

| Type                                 | Usage                                                                 |
|:-------------------------------------|:----------------------------------------------------------------------|
| [Data message](#data-messages)       | The number of messages of the current request or group (see [`id`](#id)) |
| [Request message](#request-messages) | The maximum number of samples to return for each requested tree node  |

The field is optional. If it is not present, then a `null` value is assumed, indicating:
- For data messages: The number of grouped messages is unknown or not provided by the sender
- For request messages: All samples in the specified time frame should be returned

Negative numbers are treated as if the value is zero.

### Depth

| Name          | `"depth"` |
|:--------------|:----------|
| Type          | Number    |
| Optional      | Yes       |
| Default value | Null      |

The `depth` field is applicable to the following messages:

| Type                                                       | Usage                                                                                    |
|:-----------------------------------------------------------|:-----------------------------------------------------------------------------------------|
| [Request message](#request-messages)                       | The number of levels below the specified path to provide in the response                 |
| [Deletion message](#data-deletion-messages)                | The number of levels below the specified path to delete                                  |
| [Subscription inclusion](#subscription-inclusion-messages) | The number of levels below the specified path to subscribe to                            |
| [Subscription exclusion](#subscription-exclusion-messages) | The number of levels below the specified path to unsubscribe from                        |
| [Subscription deletion](#subscription-deletion-messages)   | The number of levels below the specified path for which to delete the subscription rules |

The field is optional. If it is not specified, then it implies an infinite depth.
Negative numbers are treated as if the value is zero.

### End

| Name          | `"end"`   |
|:--------------|:----------|
| Type          | Number    |
| Optional      | Yes       |
| Default value | Null      |

The `end` field contains an optional timestamp, encoded as a floating-point number of the seconds since 1.1.1970 (UTC).
It signifies the end of a time interval in the following messages:

| Type                                        | Usage                                              | Default     |
|:--------------------------------------------|:---------------------------------------------------|:------------|
| [Request message](#request-messages)        | The end time of the requested interval (inclusive) | End of time |
| [Deletion message](#data-deletion-messages) | The end time of the interval to delete (inclusive) | End of time |

The `end` field is optional. If it is not present, then the default value in the table above is implied.

Example:

```json
{
    "end": 123456789.012
}
```

### Id

| Name          | `"id"`    |
|:--------------|:----------|
| Type          | Number    |
| Optional      | Yes       |
| Default value | Null      |

The `id` field can be used with the following messages:

| Type                                 | Usage                                                                                                                        |
|:-------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------|
| [Data message](#data-messages)       | The id of the request to which this message responds, or an id of a single data transmission spread across multiple messages |
| [Request message](#request-messages) | The id of the request, to associate the responses with                                                                       |

The field is optional. If it is not present, then no id is used.
Requests without an id produce responses without an id.
Data messages without an id can be further processed without waiting for other messages.

### Children

| Name          | `"children"` |
|:--------------|:-------------|
| Type          | Object       |
| Optional      | Yes          |
| Default value | `{}`         |

The `children` field is only applicable to [Data messages](#data-messages) and can be used to provide additional samples beneath the top-level [path](#path).
This allows the transmission of complex tree structures.
The content of the field is a JSON object, where each field name corresponds to the name of a child, and the value is another JSON object:

```json
{
    "children": {
        "child1": {

        },
        "child2": {

        }
    }
}
```

Each object in this nested structure represents a node in a tree, and can have the following fields:

| Field      | Description                                                                           |
|:-----------|:--------------------------------------------------------------------------------------|
| `value`    | A value associated with this node, see [value field](#value)                          |
| `type`     | The type of the value, see [value type](#value-type) (Note the different field name!) |
| `time`     | The timestamp associated with the value, see [time](#time)                            |
| `samples`  | An array of samples for this node, see [samples](#samples)                            |
| `children` | Additional child nodes of this node                                                   |

The same rules as for the top level apply, e.g. a `time` without a `value` is ignored.
A node without `value` and `type` has no sample at its own level.
Other fields in child nodes are ignored.

This recursive definition allows the transmission of complex tree structures, with multiple levels of nesting.

## Decoding rules

The following rules apply to all messages:

- **Field order:** The order of fields in an object is not significant, e.g. `type` may appear before or after `value`, and `message` may appear anywhere in the object.
- **Unknown fields:** Fields not defined in this document are ignored, as are fields that don't belong in an object (e.g. `path` in a child node), and data fields in messages other than data messages.
  Message fields that are currently unused by a message type are still decoded (see [message types](#message-types)).
- **Numbers:** Numbers are decoded in a best effort manner, since other systems may support a different range or precision:
  - Integers are 64-bit signed values. Integer values outside of this range are clamped to the smallest or largest value.
    Integers with a fractional part or exponent are accepted if they represent a whole number, e.g. `3.0` or `1e2`.
  - Doubles are 64-bit IEEE values. Numbers too large for a double are decoded as infinity, and precision may be lost.
  - `count` and `depth` are clamped in the same way as integer values, and negative values are treated as zero.
  - Note that many JSON implementations (e.g. JavaScript) represent all numbers as doubles,
    so integers with a magnitude larger than 2^53 may lose precision when processed by such systems.
- **Duplicate fields:** If a field appears multiple times in an object, then the last occurrence is used.
- **Booleans:** Booleans are only valid as values, where they are decoded as integers (see [Value](#value)).

## Examples

Consider the following information:
```
rover.position.x = 12.3
rover.position.y = 23.4
rover.position.x.target = 15.0
rover.velocity.x = 34.5
rover.velocity.y = 45.6
```

Encoded in a single message:

```json
{
    "path": ["rover"],
    "children": {
        "position": {
            "children": {
                "x": {
                    "value": 12.3,
                    "children": {
                        "target": {
                            "value": 15.0
                        }
                    }
                },
                "y": {
                    "value": 23.4
                }
            }
        },
        "velocity": {
            "children": {
                "x": {
                    "value": 34.5
                },
                "y": {
                    "value": 45.6
                }
            }
        }
    }
}
```

It's also possible to encode multiple samples for each node, and include timestamps as needed.

## Value encoding

Single values can also be encoded independently of messages, e.g. to store them in files or databases.
The same representation as for values within messages is used.
Two variants exist:

### Tagged values

A tagged value contains its type, and is encoded as a JSON array with two elements: The [type](#value-type) and the encoded [value](#value).
This is the same representation as a single element of a [list](#list).

| Value                   | Encoding                              |
|:------------------------|:--------------------------------------|
| `Null`                  | `["null",null]`                       |
| `Int(5)`                | `["integer",5]`                       |
| `Double(1.5)`           | `["double",1.5]`                      |
| `Double(NaN)`           | `["double","nan"]`                    |
| `String("x")`           | `["string","x"]`                      |
| `Data(0x01 0x02 0x03)`  | `["data","AQID"]`                     |
| `Path("a", "b")`        | `["path",["a","b"]]`                  |
| `List(Int(1), "x")`     | `["list",["integer",1,"string","x"]]` |
| `Map("a" -> Int(1))`    | `["map",["a","integer",1]]`           |

The array must contain exactly two elements, and no other data may follow it.

### Value content

If the type of a value is known from another source (e.g. a database column), then only the content of the value can be encoded, without its type.
The content is the encoded [value](#value) itself, e.g. `5` for an integer, `"AQID"` for data, or `["integer",1,"string","x"]` for a list.
The type must be provided when decoding, so the content is always decoded as the given type,
e.g. `"nan"` is decoded as NaN for the type `double`, and `"a"` as a path for the type `path`.
