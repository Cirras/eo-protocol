# Elements
A specification of the XML elements used within `protocol.xml` files.

## Glossary

| element                          | description                                                                                                                                           |
|----------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|
| \<protocol>                      | The root element of a protocol document.                                                                                                              |
| [\<enum>](#the-enum-element)     | Defines an enumeration type.                                                                                                                          |
| [\<value>](#the-value-element)   | Defines a value in an `enum`.                                                                                                                         |
| [\<struct>](#the-struct-element) | Defines an object type.                                                                                                                               |
| [\<packet>](#the-packet-element) | Defines a packet type. Similar to a `struct`, but a `packet` is mapped to a packet `family` and `action`.                                             |
| [\<field>](#the-field-element)   | Defines a field in a `struct` or `packet`.                                                                                                            |
| [\<array>](#the-array-element)   | Defines an array field in a `struct` or `packet`.                                                                                                     |
| [\<dummy>](#the-dummy-element)   | Defines a dummy data field in a `packet`. Similar to an unnamed `field`, but a `dummy` should only be written if the packet would otherwise be empty. |
| [\<switch>](#the-switch-element) | Defines a section in a `struct` or `packet` where the structure may differ based on the value of a field.                                             |
| [\<case>](#the-case-element)     | Defines an optional portion of a `struct` or `packet` based on the value of the field specified in the parent `switch`.                               |
| \<chunked>                       | Defines a section of a `struct` or `packet` where [chunked reading](chunks.md) is enabled.                                                            |
| [\<break>](#the-break-element)   | Defines a break in a `chunked` section of a `struct` or `packet`.                                                                                     |
| \<comment>                       | Documentation for a particular element in the protocol document, with code/docs generation in mind.                                                   |

## The \<enum> Element

| attribute | description                                                                                                                          |
|-----------|--------------------------------------------------------------------------------------------------------------------------------------|
| name      | The name of the enumeration type.<br>`PascalCase` should be used.                                                                    |
| type      | The underlying type of the enumeration.<br>For more information, see documentation on [underlying types](types.md#underlying-types). |
| clamp     | If **true**, then an out-of-bounds value should be clamped to the nearest expected value.                                            |

> **Implementation notes**
> - An "out-of-bounds" value is smaller than the smallest possible value, or larger than the largest possible value.
> - A value that's *within* the possible range but undefined in the enumeration should be considered unknown.

## The \<value> Element
Text content is required and must be a numeric value.

| attribute | description                                                                                                                   |
|-----------|-------------------------------------------------------------------------------------------------------------------------------|
| name      | The name of the enumeration value.<br>`PascalCase` should be used.                                                            |
| default   | If **true**, then an unknown value should be treated as this value instead.<br>Only one enumeration value can be the default. |

> **Implementation notes**
> - An unknown value which is not handled by `clamp` or `default` should be considered a deserialization error.

## The \<struct> Element

| attribute | description                                                  |
|-----------|--------------------------------------------------------------|
| name      | The name of the object type.<br>`PascalCase` should be used. |

## The \<packet> Element

| attribute | description                                  |
|-----------|----------------------------------------------|
| family    | A value from the `PacketFamily` enumeration. |
| action    | A value from the `PacketAction` enumeration. |

> **Implementation notes**
> - The `family`/`action` pair forms the identifier for each packet.
> - Duplicate `family`/`action` pairs in the same file should be considered an error.

## The \<field> Element
Text content is optional and specifies a hardcoded field value.

| attribute     | description                                                                                            |
|---------------|--------------------------------------------------------------------------------------------------------|
| name          | The name of the field.<br>`snake_case` should be used.                                                 |
| type          | The type of the field.<br>For more information, see documentation on [types](types.md).                |
| length        | The length of the field.<br>The value should either be a numeric literal or the name of another field. |
| length-offset | An offset that will be applied to the value of `length`.<br>The value should be a numeric literal.     |
| padded        | If **true**, the field is padded with `0xFF` bytes.                                                    |
| optional      | If **true**, the field is considered optional for the purposes of data serialization.                  |

> **Implementation notes**
> - If `name` is not provided...
>   - text content must be provided.
>   - neither getter nor setter should be generated for the field value.
> - If text content is provided...
>   - a setter should not be generated for the field value.
>   - the field type must be one of the [basic types](types.md#basic-types).
>   - `length` must not be a field reference.
> - `length`, `length-offset`, and `padded` are only allowed for string types.
> - If `padded` is **true**...
>   - `length` must be provided.
>   - The serializer should pad the string with `0xFF` bytes up to the specific `length` (before encoding, if applicable).
>   - The deserializer should treat `0xFF` as a terminator (after decoding, if applicable).
> - If `optional` is **true**...
>   - `name` must be provided.
>   - An unset field value should not be written by the serializer.
>   - Non-optional fields are forbidden afterwards.
>     - This only applies within the current chunk if [chunked reading](chunks.md) is enabled.

## The \<array> Element

| attribute     | description                                                                                                    |
|---------------|----------------------------------------------------------------------------------------------------------------|
| name          | The name of the array field.<br>`snake_case` should be used.                                                   |
| type          | The element type of the array.<br>For more information, see documentation on [types](types.md).                |
| length        | The length of the array.<br>The value should either be a numeric literal or the name of another field.         |
| length-offset | An offset that will be applied to the value of `length`.<br>The value should be a numeric literal.             |
| optional      | If **true**, the field is considered optional for the purposes of data serialization.                          |
| delimited     | If **true**, the elements of the array will be delimited with `0xFF` break bytes (without trailing delimiter). |

> **Implementation notes**
> - If `optional` is **true**...
>   - An unset field value should not be written by the serializer.
>   - Non-optional fields are forbidden afterwards.
>     - This only applies within the current chunk if [chunked reading](chunks.md) is enabled.
> - `delimited` is only allowed if the array is within a `<chunked>` element.
> - If `delimited` is **false**, then the element type must have a fixed size.
> - If `delimited` is **false** _and_ `length` is not provided...
>   - The array must be the final field in the data structure (or in the chunk, if [chunked reading](chunks.md) is enabled)
>   - Assuming integer division, the number of elements can be calculated as `remaining_length / element_size`.

## The \<dummy> Element
Text content is required and specifies a hardcoded dummy value.

| attribute     | description                                                                                            |
|---------------|--------------------------------------------------------------------------------------------------------|
| type          | The type of the field.<br>For more information, see documentation on [types](types.md).                |

> **Implementation notes**
> - The official EO client and server never send empty packets.
>   - When a packet would otherwise be empty, a dummy value is written instead.
>   - These are typically single-byte strings or numbers.
> - `type` must be one of the [basic types](types.md#basic-types).
> - `dummy` elements must not be followed by any other elements.
> - If the packet ends up containing data (e.g. an optional field was provided), the dummy data should not be written by the serializer.

## The \<switch> Element

| attribute     | description                                                                                                               |
|---------------|---------------------------------------------------------------------------------------------------------------------------|
| field         | The name of a field in the data structure. The referenced field's value determines which child `case` element is matched. |

> **Implementation notes**
> - The referenced field must be unconditionally present (it must not be optional, or within a `switch` element).
> - The referenced field must have a numeric or enumeration type.

## The \<case> Element

| attribute     | description                                                                                                                                                        |
|---------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| value         | The field value required for this switch case to be matched.                                                                                                       |
| default       | If **true**, then this switch case will be considered active if the field value does not match any other `case` elements. Only one switch case can be the default. |

> **Implementation notes**
> - If `default` is true...
>   - `value` cannot be specified.
>   - the case must appear at the end of the switch.

## The \<break> Element

This element has no attributes.

> **Implementation notes**
> - A `break` is a raw `0xFF` byte, which has special meaning when [chunked reading](chunks.md) is enabled.
> - `<break>` elements are not allowed outside of `<chunked>` elements.