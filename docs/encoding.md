# Encoding Routines
Pseudo-code for the encoding routines used with encoded integer and string types.

## Numeric limits
```c
CHAR_MAX = 253;
SHORT_MAX = CHAR_MAX * CHAR_MAX;
THREE_MAX = CHAR_MAX * CHAR_MAX * CHAR_MAX;
INT_MAX = CHAR_MAX * CHAR_MAX * CHAR_MAX * CHAR_MAX;
```

> **Note:**
>  - The largest valid value for each type is `TYPE_MAX - 1`.

## Numbers

### Encode
```c
EncodeNumber(number)
{
    value = number;

    d = 0xFE;
    if (number >= THREE_MAX)
    {
        d = value / THREE_MAX + 1;
        value %= THREE_MAX;
    }

    c = 0xFE;
    if (number >= SHORT_MAX)
    {
        c = value / SHORT_MAX + 1;
        value %= SHORT_MAX;
    }

    b = 0xFE;
    if (number >= CHAR_MAX)
    {
        b = value / CHAR_MAX + 1;
        value %= CHAR_MAX;
    }

    a = value + 1;

    return {a, b, c, d};
}
```

### Decode
```c
GetByteValue(byte)
{
    if (byte === 254)
    {
        byte = 1;
    }

    --byte;

    return byte;
}

DecodeNumber(a = 0xFE, b = 0xFE, c = 0xFE, d = 0xFE)
{
    a = GetByteValue(a);
    b = GetByteValue(b);
    c = GetByteValue(c);
    d = GetByteValue(d);

    return d * THREE_MAX + c * SHORT_MAX + b * CHAR_MAX + a;
}
```

## Strings
The core logic is shared between string encoding and decoding routines, with the only difference between them being a symmetrical reverse.

### Invert Characters
```c
InvertCharacters(bytes)
{
    flippy = (bytes.length is odd);

    for (i = 0; i < bytes.length; ++i)
    {
        c = bytes[i];
        f = 0;

        if (flippy)
        {
            f = 0x2E;
            if (c >= 0x50)
            {
                f *= -1;
            }
        }

        if (c >= 0x22 && c <= 0x7E)
        {
            bytes[i] = 0x9F - c - f;
        }

        flippy = !flippy;
    }
}
```

### Encode
```c
EncodeString(bytes)
{
    InvertCharacters(bytes);
    Reverse(bytes);
    return bytes;
}
```

### Decode
```c
DecodeString(bytes)
{
    Reverse(bytes);
    InvertCharacters(bytes);
    return bytes;
}

```