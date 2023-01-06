# Types
A specification of the data types that can be referenced in protocol files.

## Basic types

| type           | size                                                                                                                            | description                                                       |
|----------------|---------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------|
| byte           | 1                                                                                                                               | An unencoded integer value.                                       |
| char           | 1                                                                                                                               | An encoded integer value.                                         |
| short          | 2                                                                                                                               | An encoded integer value.                                         |
| three          | 3                                                                                                                               | An encoded integer value.                                         |
| int            | 4                                                                                                                               | An encoded integer value.                                         |
| bool           | 1                                                                                                                               | An encoded integer value that's interpreted as `true` or `false`. |
| string         | variable or fixed `length` from a [\<field>](elements.md#the-field-element) element.<br>unbounded if `length` is not specified. | An unencoded string value.                                        |
| encoded_string | variable or fixed `length` from a [\<field>](elements.md#the-field-element) element.<br>unbounded if `length` is not specified. | An encoded string value.                                          |

> **Implementation notes:**
> - The `bool` type is interpreted as:
>   - `false` for a value of 0.
>   - `true` for any other value.
> - See [encoding](encoding.md) for more information on the encoding routines used for the basic types.

## Array types
- Defined by [\<array>](elements.md#the-array-element) elements.
- Their size is **variable** if any of the following are true:
    - the element type has variable size.
    - the `length` attribute is not a numeric literal.
    - the `delimited` attribute is true.
        - It's possible to omit data or insert garbage data at the end of each chunk.
        - See the documentation on [chunked reading](chunks.md) for more information.
- Their size is **unbounded** if any of the following are true:
    - the element type has unbounded size.
    - the `length` attribute is not provided.
- If applicable, **fixed** size can be calculated as `element_size * length`.

## Custom types

### Enumerations
- Defined by [\<enum>](elements.md#the-enum-element) elements.
- Their size is specified by the underlying type.

### Structs
- Defined by [\<struct>](elements.md#the-struct-element) elements.
- Their size is **variable** when any of the following are present:
    - `field` where any of the following are true:
        - the type is variable-sized.
        - the `optional` attribute is true.
    - `array` where any of the following are true:
        - variable-sized according to the conditions listed [above](#array-types).
        - the `optional` attribute is true.
    - `dummy` where the type is variable-sized.
    - `chunked`
        - It's possible to omit data or insert garbage data at the end of each chunk.
        - See the documentation on [chunked reading](chunks.md) for more information.
     - `switch`
- Their size is **unbounded** when any of the following are present:
    - `field` with unbounded type, which is not followed by a `break`.
    - `array` with unbounded size according to the conditions listed [above](#array-types), which is not followed by a `break`.
    - `dummy` with unbounded type.
- If applicable, the **fixed** size can be calculated by adding the length of all fields.

## Underlying types
- An underlying type specifies how a type will be treated for serialization purposes.
  - **Example:** The `FileType` enum has an underlying type of `char`, so it will be serialized and deserialized as a single-byte encoded integer.
- Only enumeration and `bool` types have underlying types.
  - Enumeration types specify their default underlying type on the [\<enum>](elements.md#the-enum-element) element via the `type` attribute.
  - `bool` types have a default underlying type of `char`.
- Underlying types can be overriden with a special `type:underlying_type` syntax in type references.
  - **Example:** `bool:short` specifies a `bool` with an underlying type of `short`.
- Only numeric types are allowed to be underlying types.
