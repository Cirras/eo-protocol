# Types
A specification of the data types that can be referenced in protocol files.

## Basic types

| type           | size                                                                                  | description                                                       |
|----------------|---------------------------------------------------------------------------------------|-------------------------------------------------------------------|
| byte           | 1                                                                                     | An unencoded integer value.                                       |
| char           | 1                                                                                     | An encoded integer value.                                         |
| short          | 2                                                                                     | An encoded integer value.                                         |
| three          | 3                                                                                     | An encoded integer value.                                         |
| int            | 4                                                                                     | An encoded integer value.                                         |
| bool           | 1                                                                                     | An encoded integer value that's interpreted as `true` or `false`. |
| string         | variable, or fixed `length` from a [\<field>](elements.md#the-field-element) element. | An unencoded string value.                                        |
| encoded_string | variable, or fixed `length` from a [\<field>](elements.md#the-field-element) element. | An encoded string value.                                          |

> **Implementation notes:**
> - The `bool` type is interpreted as:
>   - `false` for a value of 0.
>   - `true` for any other value.
> - See [encoding](encoding.md) for more information on the encoding routines used for the basic types.

## Array types
- Defined by [\<array>](elements.md#the-array-element) elements.
- They are variable-sized if any of the following are true:
    - the `length` attribute is not provided.
    - the `delimited` attribute is true.
        - It's possible to omit data or insert garbage data at the end of each chunk.
        - See the documentation on [chunked reading](chunks.md) for more information.
- Fixed size can be calculated as `element_size * length`.

## Custom types

### Enumerations
- Defined by [\<enum>](elements.md#the-enum-element) elements.
- Their size is specified by the base type.

### Structs
- Defined by [\<struct>](elements.md#the-struct-element) elements.
- They are variable-sized when any of the following are present:
    - `field` where any of the following are true:
        - the type is variable-sized.
        - the `optional` attribute is true.
    - `array` where any of the following are true:
        - variable-sized according to the conditions listed [above](#array-types).
        - the `optional` attribute is true.
    - `chunked`
        - It's possible to omit data or insert garbage data at the end of each chunk.
        - See the documentation on [chunked reading](chunks.md) for more information.
     - `switch`
- Fixed size can be calculated by adding the length of all fields.